---
title: kafka常见面试题
date: 2022-08-16 14:48:37
tags: 面试题
categories: kaaka
---

### kafka和RabbitMq的对比

1. 应用场景上：

   - RabbitMQ,遵循AMQP协议，用在实时的对可靠性要求比较高的消息传递上。
   - kafka它主要用于处理活跃的流式数据,大数据量的数据处理上。

2. 在架构模型上：

   - RabbitMQ遵循AMQP协议，RabbitMQ的broker由Exchange,Binding,queue组成。其中exchange和binding组成了消息的路由键；客户端Producer通过连接channel和server进行通信，Consumer从queue获取消息进行消费（长连接，queue有消息会推送到consumer端，consumer循环从输入流读取数据）。rabbitMQ以broker为中心；有消息的确认机制。
   - kafka遵从一般的MQ结构，producer，broker，consumer，以consumer为中心，消息的消费信息保存的客户端consumer上，consumer根据消费的点，从broker上批量pull数据；无消息确认机制。

3. 吞吐量上：

   - rabbitMQ在吞吐量方面稍逊于kafka，他们的出发点不一样，rabbitMQ支持对消息的可靠的传递，支持事务，不支持批量的操作
   - kafka具有高的吞吐量，内部采用消息的批量处理，zero-copy机制，数据的存储和获取是本地磁盘顺序批量操作，具有O(1)的复杂度，消息处理的效率很高。

4. 在可用性方面:

   - rabbitMQ支持miror的queue，主queue失效，miror queue接管。
   - kafka的broker支持主备模式。

5. 在集群负载均衡方面:

   - rabbitMQ的负载均衡需要单独的loadbalancer进行支持。
   - kafka采用zookeeper对集群中的broker、consumer进行管理，可以注册topic到zookeeper上；通过zookeeper的协调机制，producer保存对应topic的broker信息，可以随机或者轮询发送到broker上；并且producer可以基于语义指定分片，消息发送到broker的某分片上。

6. 消息的删除时间上：

   1. rabbitMQ 消费完了就删除。

   2. kafka一般不会删除消息，不管这些消息有没有被消费。只会根据配置的日志保留时间(log.retention.hours)确认消息多 久被删除，默认保留最近一周的日志消息。这些日志可以被重复读取和无限期保留。

      

### 为什么要对Topic下数据进行分区存储？

1. commit log文件会受到所在机器的文件系统大小的限制，分区之后可以将不同的分区放在不同的机器上，相当于对 

数据做了**分布式存储**，理论上一个topic可以处理任意数量的数据。 

  2、为了**提高并行度**，consumer可以并行去处理不同分区拿到的数据。



### 怎么理解**Topic，Partition和Broker** ？

1. 一个topic，代表逻辑上的一个业务数据集，比如按数据库里不同表的数据操作消息区分放入不同topic。

2. 订单相关操作消息放入订单topic，用户相关操作消息放入用户topic，对于大型网站来说，后端数据都是海量的，订单消息很可能是非常巨量的，比如有几百个G甚至达到TB级别，如果把这么多数据都放在一台机器上可定会有容量限制问题，那么就可以在 topic内部划分多个partition来分片存储数据。

3. 不同的partition可以位于不同的机器上，每台机器上都运行一个Kafka的进程Broker。

   

### **kafka**的消费者是**pull(**拉**)**还是**push(**推**)**模式，这种模式有什么好处？

Kafka 遵循了一种大部分消息系统共同的传统的设计：producer 将消息推送到 broker，consumer 从broker 拉取消息。

优点：pull模式消费者自主决定是否批量从broker拉取数据，而push模式在无法知道消费者消费能力情况下，不易控制推送速度，太快可能造成消费者奔溃，太慢又可能造成浪费。

缺点：如果 broker 没有可供消费的消息，将导致 consumer 不断在循环中轮询，直到新消息到到达。为了避免这点，Kafka 有个参数可以让 consumer阻塞直到新消息到达，当然也可以阻塞直到消息的数量达到某个特定的量这样就可以批量发送。



## 消费进度Offset的理解

- 在消费者对指定消息分区进行消息消费的过程中，**需要定时地将分区消息的消费进度Offset记录到Zookeeper上**，以便在该消费者进行重启或者其他消费者重新接管该消息分区的消息消费后，能够从之前的进度开始继续进行消息消费。

- 现在是放在了一个叫做__consumer_offset 的topic 里，**key是** consumerGroupId+topic+分区号，value就是当前offset的值。默认是50个分区，由consumer自己管理。 

- **每个consumer是基于自己在commit log中的消费进度(offset)来进行工作的**。
- 这意味kafka中的consumer对集群的影响是非常小的，添加一个或者减少一个consumer，对于集群或者其他consumer 来说，都是没有影响的，因为每个consumer维护各自的消费offset。

- 一般情况下我们按照顺序逐条消费commit log中的消息，当然我可以通过指定offset来重复消费某些消息， 或者跳过某些消息。



## 集群消费

- leader处理所有的针对这个partition的读写请求。
- 而followers被动复制leader的结果，不提供读写(主要是为了保证多副本数据与消费的一致性）。如果这个leader失效了，其中的一个follower将会自动的变成新的leader。



## 消费顺序怎么保证？

- 一个partition同一个时刻，在一个consumer group里只能有一个consumer去消费，从而保证消费顺序。

- 发送消息的时候，往一个partition 中去发。取消息的时候也从一个partition 中去取。
- consumer group中consumer 的数量应小于分区的数量，否则多出的consumer得不到信息。
- Kafka 只能在partition范围内保证消费顺序，不能不能保证在一个topic中，多个partition的消费顺序。
- 如果有在总体上保证消费顺序的需求，那么我们可以通过将topic的partition数量设置为1，将consumer group中的 

consumer 数量也设置为1，但是这样会影响性能，所以kafka的顺序消费很少用。



#### kafka producer 中ack的配置是什么意思？

Kafka通过配置request.required.acks属性来确认消息的生产：

- 0 生产者将数据发送出去就不管了，不去等待任何返回。这种情况下数据传输效率最高，但是数据可靠性确是最低的，存在丢消息：数据还没写入leader，leader就挂了
- 1（默认） 数据发送到Kafka后，经过leader成功接收消息的的确认，就算是发送成功了。在这种情况下，如果leader宕机了，则会丢失数据。因为follo wer还没同步数据。
- -1或all producer需要等待ISR中的所有follower都确认接收到数据后才算一次发送完成，可靠性最高。当ISR中所有Replica都向Leader发送ACK时，leader才commit，这时候producer才能认为一个请求中的消息都commit了。这种性能不高，一般是金融级别或者和钱打交道的时候采用。

**避免消息丢失就可以将ack设置成-1或all**



### kafka 如何不消费重复数据？

重复数据的产生：

- 发送方：发送消息如果配置了重试机制，比如网络抖动时间过长导致发送端发送超时，实际broker可能已经接收到消息，但发送方会重新发送消息

- 消费方：如果消费这边配置的是自动提交，刚拉取了一批数据处理了一部分，但还没来得及提交，服务挂了，下次重启又会拉取相同的一批数据重复处理。

 解决方式：

设置一个唯一的id去标志这条消息，消费时去查询一下，如果已经消费过了，则不进行消费。



#### Zookeeper 在 Kafka 中的作用?

Zookeeper 主要用于在集群中不同节点之间进行通信

- 在Zookeeper上会有一个专门**用来进行Broker服务器列表记录**的节点/brokers/ids

- 每个Broker在启动时，都会到Zookeeper上进行注册，即到/brokers/ids下创建属于自己的节点，如/brokers/ids/[0...N]。

- Kafka使用了全局唯一的数字来指代每个Broker服务器，不同的Broker必须使用不同的Broker ID进行注册，创建完节点后，**每个Broker就会将自己的IP地址和端口信息记录**到该节点中去。其中，Broker创建的节点类型是临时节点，一旦Broker宕机，则对应的临时节点也会被自动删除。





