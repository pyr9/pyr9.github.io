---
title: RabbitMQ的高级特性
date: 2022-09-10 22:17:03
tags: 消息中间件
categories: RabbitMQ
---

# RabbitMQ的高级特性

## **如何保证消息的顺序？**

消息的顺序是指的是，一个组内消息的顺序，而不是整个mq里的消息顺序。比如一个下单过程需要完成扣款，减库存，通知快递发货，这一组消息的顺序不能乱。

- 发送端：一组有序消息，只发到一个队列中，利用队列的FIFO特性保证消息在发送时顺序不会乱。

  如果发送端配置了重试机制，就可能出现发送方发送时是1，2，3，但1发送失败，重试发送1，这样收到的消息就是2，3，1。这种情况下，需要同步的去发消息，只有第一个消息发送成功了，再去发送2，3。

- 消费者端：进行消费时，保证只有一个消费者，同时指定prefetch属性为1，即每次RabbitMQ都只往消费端推送一个消息。这样可以保证消费时的有序。

**显然，这是以极度消耗性能作为代价的，在实际适应过程中，应该尽量避免这种场景。**



## 消息积压和解决

### 1.发送方发送流量太大

- 降低消息生产的速度。生产者端产生消息的速度通常是跟业务息息相关的，一般情况下不太好直接优化。但是可以采用批量发送消息的方式，降低IO频率。

### 2. 消费者能力不足

- 修改消费端程序，写一个临时的分发数据的 consumer 程序，将收到的消息快速转发给临时建立好原先 10 倍的 queue， 接着临时征用 10 倍的机器来部署 consumer，每一个 consumer 消费一个临时 queue 的数据。

- 上线专门的消费者组。专门用来将消息快速转录。保存到数据库或者Redis，然后再慢慢进行处理。

- 对于单个消费者端，可以通过配置提升消费者端的吞吐量。例如

  ```java
  # 单次推送消息数量
  spring.rabbitmq.listener.simple.prefetch=1
  # 消费者的消费线程数量
  spring.rabbitmq.listener.simple.concurrency
  ```

### 3.消费者有bug或者宕机了

- 将消费不成功的消息转发到其他队列里，类似死信队列，后面再慢慢分析。kafka没有死信队列需要自己实现。

#### 其他解决方式

- RabbitMQ服务端：尝试使用新推出的Quorum以及Stream队列，也能很好的提高服务端的消息堆积能力。但是这两种队列的稳定性和周边生态都还不够完善。
- RabbitMQ一直以来，就是对于消息堆积问题的处理不好。当RabbitMQ中有大量消息堆积时，整体性能会严重下降。不行就换kafka



## MQ中消息失效

RabbtiMQ 是可以设置过期时间 的，也就是 TTL。如果消息在 queue 中积压超过一定的时间就会被 RabbitMQ 给清理掉，这个数据就没了。

解决：批量重导。就是大量积压的时候，我们当时就 直接丢弃数据了，然后等过了高峰期以后。将丢失的那批数据，写个临时程序，一点一点的查出来，然后重新灌入 mq 里面去

## 消息的幂等性

造成消息重复的根本原因是：网络不可达。 

所以解决这个问题的办法就是绕过这个问题。那么问题就变成了：如果消费端收 到两条一样的消息，应该怎样处理？ 

方案一： 利用一张日志表来记录已经处理成功的消息的 ID，如 果新到的消息 ID 已经在日志表中，那么就不再处理这条消息。 

方案二：消息生成唯一的id，存储在redis中。

方案二：

* 第一次 执行更新语句的是一样，version =1 

  ```
  update account set price = price -100, version = version + 1, where id = 1 and version = 1
  ```

- 第二次, 执行更新语句的是一样，version 已经变成了2，此时找`where version = 1 ` 就无法找到

  ```
  update account set price = price -100, version = version + 1, where id = 1 and version = 1
  ```



##  消费端限流

* 在<rabbit:listener-container> 中配置`prefetch`属性设置消费端一次拉取多少消息
* 消费端的确认模式需要是手动确认



## TTL

1. TTL全称：time to live 消息存活时间或消息过期时间
2. 消息达到了存活时间后，如果还没被消费，会被自动移除
3. RabbitMQ可以对消息设置过期时间，也可以对整个队列设置过期时间。



## 死信队列

* 当消息成为死信后，可以被重新发送到一个交换机，这个交换机就是死信交换机，它绑定的队列就是死信队列

* 成为死信的条件：

  * 消息达到了存活时间，还没有被消费。
  * 消费者拒收消息，并且不重回队列。  
  * 队列到达了指定的长度限制

* 代码

  - producer:

    ```java
    # 指定了消息的过期时间为10s 
    AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
                    .deliveryMode(2)
                    .contentEncoding("UTF-8")
                    .expiration("10000")
                    .build();
      channel.basicPublish(exchange, routingKey, true, properties, msg.getBytes());
    ```

  - consumer:

    ```java
    String exchangeName = "test_dlx_exchange";
    channel.exchangeDeclare(exchangeName, "topic", true, false, null);
    
    # 在队列加上一个参数，指定死信队列
    Map<String, Object> agruments = new HashMap<String, Object>();
    agruments.put("x-dead-letter-exchange", "dlx.exchange");
    channel.queueDeclare(queueName, true, false, false, agruments);
    ```



## 延迟队列 

延迟队列，消息进入队列后，不会立即被消费，而是等到一定的时间，才会被消费。

使用场景：

1. 用户下单后，30分钟未支付，取消订单，回滚库存
2. 新用户注册7天后，发送短信问候

> 当然上面的场景也可以用定时器实现

rabbitmq现在不支持延迟队列，延迟队列的实现需要借助TTL和死信队列。具体实现流程：

* 用户下单，把消息发送到Queue1中，不设置Consumer1，设置Queue1队列里的消息存活时间为30分钟，等待30分钟后，消息成为死信。
* 死信的消息发送到Queue2，添加Consumer2监听Queue2

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gwld73gtjaj31ga0o440f.jpg" style="zoom:43%;" />



#### 死信队列和延时队列的区别

* 死信队列，监听的是Queue1,成为死信的消息会被丢到DLX中，或者不处理自己清理掉
* 延迟队列，监听的是死信队列

