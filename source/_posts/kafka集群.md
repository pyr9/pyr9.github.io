---
title: kafka集群
date: 2022-08-11 20:52:51
tags: 
 - 消息队列
 - 集群
 - 分布式框架
categories: kafka
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





