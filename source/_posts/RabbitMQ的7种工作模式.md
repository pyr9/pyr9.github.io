---
title: RabbitMQ的7种工作模式
date: 2021-10-27 21:35:27
tags: 消息中间件
categories: RabbitMQ
---

# **RabbitMQ的7种工作模式** 

## 1. simple模式

![image-20230228223058625](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228223058625.png)

- 最简单的收发模式。生产者发送一个消息到一个指定的queue，中间不需要任何exchange规则。消费者端通过queue方式进行消费。

- 代码

  - producer:

    ```java
    channel.queueDeclare(QUEUE_NAME, true, false, false, null);
    String message = "hello world";
    channel.basicPublish("", QUEUE_NAME, null, message.getBytes(StandardCharsets.UTF_8));
    ```

  - consumer:

    ```java
    channel.queueDeclare(QUEUE_NAME, true, false, false, null);
    ```

    

## 2. work工作模式(资源的竞争)

![image-20230228223108418](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228223108418.png)

* 一个生产者，多个消费者。

* consumer1, consumer2 同时监听同一 个队列,消息被消费。服务器根据负载方案决定把消息发给一个指定的consumer处理。work主要有两种模式：

  >- **轮询分发**：一个消费者消费一条，**按均分配**，woek模式下默认是采用轮询分发方式。比较简单，比如生产费者发送了6条消息到队列中，如果有3个消费者同时监听着这一个队列，那么这3个消费者每人就会分得2条消息。
  >- 公平分发**：根据消费者的消费能力进行公平分发，处理得快的分得多，处理的慢的分得少，**能者多劳。

* 隐患：高并发情况下,默认会产生某一个消息被多个消费者共同使用,可以设置一个开关(syncronize) 保证一条消息只能被一个消费者使用。 



## 3. publish/subscribe发布订阅(共享资源)

![](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228223121109.png)

-  type为**"fanout"** 的exchange

- 每个消费者监听自己的队列； 生产者将消息发给broker，由交换机将消息转发到绑定此交换机的每个队列，每个绑定交换机的队列都将接收到消息。

- 代码

  - producer:

    ```java
    channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.FANOUT);
    channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes(StandardCharsets.UTF_8));
    ```

  - consumer:

    ```java
    channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
    channel.queueDeclare("queue1", false, false, false, null);
    channel.queueBind("queue1", EXCHANGE_NAME, "");
    ```

## **4. Routing路由模式** 

![image-20230228223131996](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228223131996.png)

- type为”**direct**” 的exchange

- 根据routing_key 进行匹配，生产者，发送消息的时候会制定routing_key，交换机根据routing_key，去匹配绑定改routing_key的队列,只能匹配上路由key对应 的消息队列,对应的消费者才能消费消息

- 代码

  - producer:

    ```java
    channel.exchangeDeclare(DIRECT_EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
    channel.basicPublish(DIRECT_EXCHANGE_NAME, ROUTING_KEY, null,message.getBytes(StandardCharsets.UTF_8));
    ```

  - consumer:

    ```java
    channel.exchangeDeclare(DIRECT_EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
    channel.queueDeclare(DIRECT_QUEUE_NAME, false, false, false, null);
    channel.queueBind(DIRECT_QUEUE_NAME, DIRECT_EXCHANGE_NAME, ROUTING_KEY);
    ```



## **5.topic 主题模式(路由模式的一种)** 

![](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228223142678.png)

- type为"**topic**" 的exchange

- 交换机根据key的规则模糊匹配到对应的队列，由队列的监听消费者接收消息消费。类似sql的模糊匹配

- \* 代表一个具体的单词。# 代表0个或多个单词

- 代码

  - producer:

    ```java
    channel.exchangeDeclare(TOPIC_EXCHANGE, BuiltinExchangeType.TOPIC);
    String message = "hello world topic!";
    channel.basicPublish(TOPIC_EXCHANGE, "hello.info", null, message.getBytes(StandardCharsets.UTF_8));
    ```

  - consumer:

    ```java
    static final String ROUTING_KEY = "*.info";
    channel.exchangeDeclare(TOPIC_EXCHANGE, BuiltinExchangeType.TOPIC);
    channel.queueDeclare(TOPIC_QUEUE_1, false, false, false, null);
    channel.queueBind(TOPIC_QUEUE_1, TOPIC_EXCHANGE, ROUTING_KEY);
    ```



## 6. RPC模式(路由模式的一种)

![image-20230228223151956](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228223151956.png)

实现不同服务间的远程调用。 springCloud feign



## 7. Publisher Confirms **发送者消息确认**

- 这个模块就是通过给发送者提供一些确认机制，来保证这个消息发送的过程是成功的

- 发送者确认模式默认是不开启的，所以如果需要开启发送者确认模式，需要手动在channel中进行声明

   `channel.confirmSelect() `

- 在官网的示例中，重点解释了三种策略

  - **1. 发布单条消息**

    channel.waitForConfirmsOrDie(5_000);这个方法就会在channel端等待RabbitMQ给出一个响应，用来表明这个消息已经正确发送到了RabbitMQ服务端。但是要注意，这个方法会同步阻塞channel，在等待确认期间，channel将不能再继续发送消息，也就是说会明显降低集群的发送速度即吞吐量。

    ```java
    while (thereAreMessagesToPublish()) {
        byte[] body = ...;
        BasicProperties properties = ...;
        channel.basicPublish(exchange, queue, properties, body);
        // uses a 5 second timeout
        channel.waitForConfirmsOrDie(5_000);
    }
    ```

    

  - **2. 发送批量消息**

    之前单条确认的机制会对系统的吞吐量造成很大的影响，所以稍微中和一点的方式就是发送一批消息后，再一起确认。

    ```java
    int batchSize = 100;
    int outstandingMessageCount = 0;
    while (thereAreMessagesToPublish()) {
        byte[] body = ...;
        BasicProperties properties = ...;
        channel.basicPublish(exchange, queue, properties, body);
        outstandingMessageCount++;
        if (outstandingMessageCount == batchSize) {
            channel.waitForConfirmsOrDie(5_000);
            outstandingMessageCount = 0;
        }
    }
    if (outstandingMessageCount > 0) {
        channel.waitForConfirmsOrDie(5_000);
    }
    ```

    

  - **3. 异步确认消息**

    Producer在channel中注册监听器来对消息进行确认。

    ```java
    channel.addConfirmListener(ConfirmCallback var1, ConfirmCallback var2); 
    ```

    发送者在发送完消息后，就会执行第一个监听器callback1，然后等服务端发过来的反馈后，再执行第二个监听器callback2



## 常用的四种交换器：

- fanout：如果交换器收到消息，将会广播到所有绑定的队列上 

- direct：如果路由键完全匹配，消息就被投递到相应的队列 

- topic：可以使来自不同源头的消息能够到达同一个队列。 使用 topic 交换器时，可以使用通配符， \* 代表一个具体的单词。# 代表0个或多个单词
- header: 不处理路由键，根据发送的消息内容中的headers属性进行匹配

## 消息的分发策略

- 消息的分发策略

    假设队列里有100条消息，有 A,B,C   3个队列

  - 发布订阅。三个队列都收到100条
  - 轮训分发。3个队列都是至少33条，剩下一条随机，不论你数据库性能怎么样，大家接受的都是公平的
  - 公平分发。根据服务器性能，去分发，哪个性能高，哪个处理的消息可能就多，能者多劳，会造成数据倾斜
  - 重发。发送消息中出现了异常后，消息没有得到应答，就会重发，kafka不支持
  - 消息拉取。就是RPC去拉取数据

  > Rabbitmq以上集中策略都支持，且是开源的
  >
  > kafka速度最快
  
  



## 中间件

- 中间件
  - 是一种应用于分布式系统的基础软件。
  - 常见的中间件：mysql，rabbit MQ
- 怎么选择中间件
  - 可以通信，跨平台。比方两个项目一个java，一个go之间要通信，就要遵循同一种协议
  - 高可用
    - 是否拥有持久化。比方中间件挂了，重启后是否可以把消息重新存储起来的能力
    - 支持集群。系统cpu不够用了，就得搭集群
  - 有分发能力，多个系统，往那个系统去发送消息

## 协议

- 网络协议三要素：
  - 语法：用户数据的结构与形式，如：http中规定了请求和响应报文的格式
  - 语义：规定了何种信息需要对应发出何种响应，如：请求get要把参数放在url中，post把参数放在body中
  - 时序：事件的执行顺序，如：先有请求后有响应

- 为什么消息中间件不用http？
  - http的请求和响应报文比较复杂，有cookie, 状态码，响应码这些，但消息中间件：只需要接受消息，存储消息，分发消息，不需要这么复杂
  - http大部分是短链接，不利于出现故障时消息持久化

- AMQP（advanced message. Queuing protocol） 高级消息队列协议
  - 采用Erlang，底层是C，速度很快
  - 特性
    - 支持分布式事务
    - 消息持久化
    - 高性能高可靠的消息处理优势

- kafka 协议
  - 基于TCP/IP的二进制协议，消息内部由长度分割，由基本数据类型构成
  - 特性
    - 结构简单
    - 解析速度快
    - 消息持久化
    - 不支持事务
