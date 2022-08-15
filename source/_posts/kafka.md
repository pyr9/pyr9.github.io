---
title: kafka集群
date: 2022-08-11 20:52:51
tags: 
 - 消息队列
 - 集群
 - 分布式框架
categories: Kafka
---

# kafka 集群

## 集群配置流程

1. 编写配置文件

   - `vim config/server.properties`

     ```
     broker.id=0     //如果是单机安装则不用修改，如果是集群安装则要保证每个broker.id配置不同的值
     listeners=PLAINTEXT://localhost:9092 
     log.dirs=/Tools/kafka_2.13-2.4.1/logs    //日志位置，该文件夹必须存在，否则启动时会报错
     zookeeper.connect=localhost:2181     //zookeeper的连接地址，多个地址用逗号分隔
     ```

2. 建立好其他2个broker的配置文件

   ```java
   cp config/server.properties config/server‐1.properties
   cp config/server.properties config/server‐2.properties
   ```

   ```
   broker.id=1     //如果是单机安装则不用修改，如果是集群安装则要保证每个broker.id配置不同的值
   listeners=PLAINTEXT://localhost:9093 
   log.dirs=/Tools/kafka_2.13-2.4.1/logs-1    //日志位置，该文件夹必须存在，否则启动时会报错
   zookeeper.connect=localhost:2181     //zookeeper的连接地址，多个地址用逗号分隔
   ```

   ```
   broker.id=2     //如果是单机安装则不用修改，如果是集群安装则要保证每个broker.id配置不同的值
   listeners=PLAINTEXT://localhost:9094 
   log.dirs=/Tools/kafka_2.13-2.4.1/logs-2    //日志位置，该文件夹必须存在，否则启动时会报错
   zookeeper.connect=localhost:2181     //zookeeper的连接地址，多个地址用逗号分隔
   ```

3. 分别启动broker实例

   ```java
   ➜  kafka_2.12-3.2.1 bin/kafka-server-start.sh -daemon config/server.properties
   ➜  kafka_2.12-3.2.1 bin/kafka-server-start.sh -daemon config/server-1.properties
   ➜  kafka_2.12-3.2.1 bin/kafka-server-start.sh -daemon config/server-2.properties
   ```

4. 查看zookeeper确认集群节点是否都注册成功

   ```java
   [zk: localhost:2181(CONNECTED) 19]  ls /brokers/ids
   [0, 1, 2]
   ```

## 集群使用

1. 创建一个新的topic，副本数设置为3，分区数设置为2： 

   ```java
   ➜  kafka_2.12-3.2.1 bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --replication-factor 3 --partitions 2 --topic test6
   ```

2. 查看下topic的情况

   ```java
   ➜  kafka_2.12-3.2.1 bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic test6
   ```

   ![](https://tva1.sinaimg.cn/large/e6c9d24ely1h55ml5a5aej21hg060myn.jpg)

3. 向新建的 test6 中发送一些message，kafka集群可以加上所有kafka节点

   ```
   ➜  kafka_2.12-3.2.1  bin/kafka-console-producer.sh ‐‐broker‐list localhost:9092,localhost:9093,localhost:9094 --topic test6  --bootstrap-server localhost:9092
   >my test msg 1
   >my test msg 2
   ```

4. 开始消费

   ```
   ➜  kafka_2.12-3.2.1 bin/kafka-console-consumer.sh --topic test6 --from-beginning --bootstrap-server localhost:9092,localhost:9093,localhost:9094
   my test msg 1
   my test msg 2
   ```

5. 测试容错性，目前broker2 是分区0的leader，现在将他kill

   ```java
   ➜  kafka_2.12-3.2.1 ps -ef|grep server‐2.properties
   ➜  kafka_2.12-3.2.1 kill -9 80024
   ```

6. 再次查看topic的情况

   ```java
   ➜  kafka_2.12-3.2.1 bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic test6
   ```

   ![](https://tva1.sinaimg.cn/large/e6c9d24ely1h56bu8415bj21h405sabk.jpg)

   可以看到，此时分区0的leader变成了broker1，在isr中也已经没有了2号节点。

  此时，依旧可以消费消息：

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h56bycarv6j21gq09cjtm.jpg)

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



#### **消费者消费消息的offset记录机制** ?

每个consumer会定期将自己消费分区的offset提交给kafka内部topic：**__consumer_offsets**，提交过去的时候，**key是** 

**consumerGroupId+topic+分区号，value就是当前offset的值**，kafka会定期清理topic里的消息，最后就保留最新的 

那条数据 

因为__consumer_offsets可能会接收高并发的请求，kafka默认给其**分配50个分区**(可以通过 offsets.topic.num.partitions设置)，这样可以通过加机器的方式抗大并发。 



#### Zookeeper 在 Kafka 中的作用?

##### 1、Broker注册

**Broker是分布式部署并且相互之间相互独立，但是需要有一个注册系统能够将整个集群中的Broker管理起来**，此时就使用到了Zookeeper。在Zookeeper上会有一个专门**用来进行Broker服务器列表记录**的节点：

每个Broker在启动时，都会到Zookeeper上进行注册，即到/brokers/ids下创建属于自己的节点，如/brokers/ids/[0...N]。

Kafka使用了全局唯一的数字来指代每个Broker服务器，不同的Broker必须使用不同的Broker ID进行注册，创建完节点后，**每个Broker就会将自己的IP地址和端口信息记录**到该节点中去。其中，Broker创建的节点类型是临时节点，一旦Broker宕机，则对应的临时节点也会被自动删除。



