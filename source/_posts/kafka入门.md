---
title: kafka入门
date: 2022-08-13 09:33:28
tags: 
 - 消息队列
 - 分布式框架
categories: kafka
---

# Kafka

## kafka 是什么？

Kafka 是一个分布式，支持分区（partition）, 多副本（replication）,基于zookeeper 的分布式消息系统。它最大的特性就是可以实时的处理大数据量。

Kafka的使用场景：

- 日志的收集：记录各种服务的log。，通过kafka以统一接口服务的方式开放给各种 consumer，例如hadoop、Hbase、Solr等。
- 消息系统：解耦和生产者和消费者、缓存消息等。
- 用户活动跟踪：Kafka经常被用来记录web用户或者app用户的各种活动，如浏览网页、搜索、点击等活动，这些活动信息被各个服务器发布到kafka的topic中，然后订阅者通过订阅这些topic来做实时的监控分析，或者装载到hadoop、数据仓库中做离线分析和掘。
- 运营指标：Kafka也经常用来记录运营监控数据。包括收集各种分布式应用的数据，生产各种操作的集中反馈，比如报警和报告。

## kafka 的整体设计

从一个较高的层面上来看，producer通过网络发送消息到Kafka集群，然后consumer来进行消费，如下图

> 服务端(brokers)和客户端(producer、consumer)之间通信通过**TCP协议**来完成。 

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h54w9l8j1hj21je0u0jy3.jpg)

还有几个概念：

- Producer: 消息生产者，向Broker发送消息的客户端 。

- Broker: 消息中间件处理节点，一个Kafka实例就是一个broker，一个或者多个Broker可以组成一个Kafka集群。

- Topic：Kafka根据topic对消息进行归类，发布到Kafka集群的每条消息都需要指定一个topic ，类似activeMq的queue

- Partition: 一个topic可以分为多个partition，每个 partition内部消息是有序的.

- ConsumerGroup:每个Consumer属于一个特定的Consumer Group，一条消息可以被多个不同的Consumer Group消费，但是一个 

  Consumer Group中只能有一个Consumer能够消费该消息

- Consumer: 消息消费者，从Broker读取消息的客户端 

## Kafka 的基本使用

1. 启动zookeeper [zookeeper特性与节点介绍 - 楼上有只喵 (panyurou.github.io)](https://panyurou.github.io/2021/09/22/zookeeper特性与节点介绍/)

2. 下载并解压： [Apache Kafka](https://kafka.apache.org/downloads)

   ```java
    cd kafka_2.12-3.2.1
   ```

3. 修改配置文件config/server.properties，根据需要修改:

   ```java
   broker.id=0     //如果是单机安装则不用修改，如果是集群安装则要保证每个broker.id配置不同的值
   log.dirs=/Tools/kafka_2.13-2.4.1/logs    //日志位置，该文件夹必须存在，否则启动时会报错
   zookeeper.connect=localhost:2181     //zookeeper的连接地址，多个地址用逗号分隔
   ```

4. 启动服务

   ```java
   bin/kafka-server-start.sh -daemon  config/server.properties
   ```

   - server.properties的配置路径是一个强制的参数
   - ­daemon表示以后台进程运行

   **我们进入zookeeper目录通过zookeeper客户端查看下zookeeper的目录树**

   ```java
   [zk: localhost:2181(CONNECTED) 2] ls /
   [admin, brokers, cluster, config, consumers, controller, controller_epoch, feature, isr_change_notification, latest_producer_id_block, log_dir_event_notification, zookeeper]
   ```

   **查看kafka节点**

   ```java
   [zk: localhost:2181(CONNECTED) 3]  ls /brokers/ids
   [0]
   ```

5. 停止服务

   ```java
   bin/kafka‐server‐stop.sh
   ```

6. 创建主题

   ```java
   ➜  kafka_2.12-3.2.1 bin/kafka-topics.sh --create --topic test --bootstrap-server localhost:9092
   Created topic test.
   ```

7. 查看kafka中目前存在的topic 

   ```java
   ➜  kafka_2.12-3.2.1  bin/kafka-topics.sh --list --bootstrap-server localhost:9092
   test
   ```

8. 发送消息

   ```java
   ➜  kafka_2.12-3.2.1 bin/kafka-console-producer.sh --topic test --bootstrap-server localhost:9092
   >my first event
   >my second event
   ```

9. 消费消息

   ```java
   ➜  kafka_2.12-3.2.1 bin/kafka-console-consumer.sh --topic test --from-beginning --bootstrap-server localhost:9092
   
   my first event
   my second event
   ```

   > --from-beginning 可选，加上的话，就可以收到历史已经发送的数据。默认不加的话，是消费最新的消息

10. 单播消费

    - 一条消息只能被某一个消费者消费的模式。类似queue模式。
    - 只需让所有消费者在同一个消费组里即可。

    分别在两个客户端执行如下消费命令，然后往主题里发送消息，结果只有一个客户端能收到消息 。

    ```java
    ➜  kafka_2.12-3.2.1 bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --consumer-property group.id=group1 --topic test
    ```

11. 多播消费

    - 一条消息能被多个消费者消费的模式，类似publish-subscribe模式。
    - 针对Kafka同一条消息只能被同一个消费组下的某一个消费者消费的特性，要实现多播只要保证这些消费者属于不同的消费组即可。
    - 我们再增加一个消费者，该消费者属于group2消费组，结果两个客户端都能收到消息 

    ```java'
    ➜  kafka_2.12-3.2.1 bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --consumer-property group.id=group2 --topic test
    ```

12. 查看消费组名

    ```java
    ➜  kafka_2.12-3.2.1 bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
    testGroup1
    group2
    group1
    ```

13. 查看消费组的消费偏移量

    ```java
    ➜  kafka_2.12-3.2.1 bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group group1
    ```

    <img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h554bsqyijj222a066jsz.jpg" style="zoom: 100%;" />

	- current-offset：当前消费组的已消费偏移量
	- log-end-offset：主题对应分区消息的结束偏移量(HW) 
	- lag：当前消费组未消费的消息数

## Topic 和 partition 详解

- topic 是一个类别的名称，同类消息，发送到一个topic下，对于每一个topic可以有多个分区(partition)。

  <img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h55j4kirf2j215i0oi0v5.jpg" style="zoom:30%;" />

- partition 是一个有序的message序列，这些message将按顺序添加到commitlog文件中。一个partition 对应一个commit log 文件，对应存放在配置的log文件中。

  ```java
  ➜  kafka_2.12-3.2.1 cd /tmp/kafka-logs
  ➜  kafka-logs cd test-0
  ➜  test-0 ls
  00000000000000000000.index     00000000000000000000.timeindex partition.metadata
  00000000000000000000.log       leader-epoch-checkpoint
  ➜  test-0 cat 00000000000000000000.log
  F_:�����6����6�(my first eventG�����������*my second event>I=;���R����R�
  ```

- 每个partion 中的消息都有一个唯一的编号，叫做offset，用来标识这条消息。

- 创建多个分区的主题

  ```java
  ➜  kafka_2.12-3.2.1 bin/kafka-topics.sh --create --topic test2 --bootstrap-server localhost:9092 ‐‐replication‐factor 1 ‐‐partitions 2
  Created topic test2.
  ```

- 查看下topic的情况

  ```java
  ➜  kafka_2.12-3.2.1 bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic test2
  ```

  <img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h556vki097j21hy04q0tz.jpg" style="zoom:150%;" />

  - leader节点负责给定partition的所有读写请求。
  - replicas 表示某个partition在哪几个broker上存在备份。不管这个几点是不是”leader“，甚至这个节点挂了，也会列出。
  - isr**(InSyncRepli)** ：leader副本保持一定同步程度的副本（包括leader）组成ISR。是replicas的一个子集。
