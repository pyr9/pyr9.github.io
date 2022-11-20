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

# 1 概念

- Kafka集群依赖于Zookeeper进行协调，Kafka节点只要注册到同一个Zookeeper上就代表它们是同一个集群的

- Kafka通过brokerId来区分集群中的不同节点

  <img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h6fdishipej211g0jstbf.jpg" style="zoom:50%;" />

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h6fdhfzd18j215u0gsach.jpg" style="zoom:50%;" />

# 2 **Kafka集群中的几个角色：**

- Broker：一般指Kafka的部署节点
- Leader：用于处理消息的接收和消费等请求，也就是说producer是将消息push到leader，而consumer也是从leader上去poll消息
- Follower：主要用于备份消息数据，一个leader会有多个follower，不提供读写(主要是为了保证多副本数据与消费的一致性）。如果这个leader失效了，其中的一个follower将会自动的变成新的leader。

# 3 kafka配置中request.required.acks

**Kafka通过配置request.required.acks属性来确认消息的生产**

- 0 生产者将数据发送出去就不管了，不去等待任何返回。这种情况下数据传输效率最高，但是数据可靠性确是最低的，存在丢消息：数据还没写入leader，leader就挂了
- 1（默认） 数据发送到Kafka后，经过leader成功接收消息的的确认，就算是发送成功了。在这种情况下，如果leader宕机了，则会丢失数据。
- -1或all producer需要等待ISR中的所有副本都确认接收到数据后才算一次发送完成，可靠性最高。当ISR中所有副本都向Leader发送ACK时，leader才commit，这时候producer才能认为一个请求中的消息都commit了。这种性能不高，一般是金融级别或者和钱打交道的时候采用。

**避免消息丢失就可以将ack设置成-1或all**

# 4 producer 的写入流程

1. producer 先从 zookeeper 的 "/brokers/.../state" 节点找到该 partition 的 leader 
2. producer 将消息发送给该 leader 
3. leader 将消息写入本地 log 
4. followers 从 leader pull  消息，写入本地 log 后 leader 发送 ACK 
5. leader 收到所有 ISR 中的 replica 的 ACK 后，增加 HW（high watermark，最后 commit 的 offset） 并向 producer 发送 ACK

# 5 Partition副本选举Leader机制

- 如果有接触过其他一些分布式组件就会了解到大部分组件都是通过投票选举来在众多节点中选举出一个leader，但在Kafka中没有采用投票选举来选举leader
- Kafka会动态维护一组Leader数据的副本（ISR）
- Kafka会在ISR列表里挑第一个broker作为leader(第一个broker最先放进ISR列表，可能是同步数据最多的副本)

Kafka有一种无奈的情况，就是ISR中副本全部宕机。对于这种情况，Kafka默认会进行unclean leader选举。Kafka提供了两种不同的方式进行处理：

1. 等待ISR中任一Replica恢复，并选它为Leader

- 等待时间较长，会降低可用性，或ISR中的所有Replica都无法恢复或者数据丢失，则该Partition将永不可用

1. 选择第一个恢复的Replica为新的Leader，无论它是否在ISR中

- 并未包含所有已被之前Leader Commit过的消息，因此会造成数据丢失，但可用性较高

# 6 Controller

在kafka集群启动的时候，会自动选举一台broker作为controller，它负责管理整个集群中所有分区和副本的状态。

-  当某个分区的leader副本出现故障时，由控制器负责为该分区**选举新的leader**副本。
- 当检测到某个分区的ISR集合发生变化时，由控制器负责**通知所有broker更新其元数据信息**。
- 当使用kafka-topics.sh脚本为某个topic**增加分区数量**时，同样还是由控制器负责让新分区被其他节点感知到。

# 7 Controller选举机制

- 集群中每个broker都会尝试在zookeeper上创建一个 /controller 临时节点
- zookeeper会保证有且仅有一个broker能创建成功，这个broker就会成为集群的总控器controller
- 当这个controller角色的broker宕机了，此时zookeeper临时节点会消失，集群里其他broker会一直监听这个临时节点，发现临时节点消失了，就竞争再次创建临时节点

# 3 集群配置流程

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





