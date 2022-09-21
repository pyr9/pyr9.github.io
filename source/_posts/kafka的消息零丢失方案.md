---
title: kafka的消息零丢失方案
date: 2022-09-13 10:05:03
tags:
- MQ
- 消息可靠性
categories: kafka
---

# Kafka如何保证消息的可靠性？

## 哪些环节会有丢消息的可能？

[RabbitMQ的消息零丢失方案 - 楼上有只喵 (panyurou.github.io)](https://panyurou.github.io/RabbitMQ的消息零丢失方案/#哪些环节会有丢消息的可能？)

## Kafka怎么保证消息可靠性?

### 1. 生产者保证消息正确发送到Kafka?

Kafka通过配置request.required.acks属性来确认消息的生产：

- 0 生产者将数据发送出去就不管了，不去等待任何返回。这种情况下数据传输效率最高，但是数据可靠性确是最低的，存在丢消息：数据还没写入leader，leader就挂了
- 1（默认） 数据发送到Kafka后，经过leader成功接收消息的的确认，就算是发送成功了。在这种情况下，如果leader宕机了，则会丢失数据。
- -1或all producer需要等待ISR中的所有副本都确认接收到数据后才算一次发送完成，可靠性最高。当ISR中所有副本都向Leader发送ACK时，leader才commit，这时候producer才能认为一个请求中的消息都commit了。这种性能不高，一般是金融级别或者和钱打交道的时候采用。

**避免消息丢失就可以将ack设置成-1或all**

> ISR**(InSyncRepli)** ：与leader保持一定同步程度的副本们（包括leader）。是replicas的一个子集

### 2. Kafka消息备份和同步

- afka通过分区的多副本策略来解决消息的备份问题。在kafka中，实现副本的目的就是冗余备份，且仅仅是冗余备份，所有的读写请求都是由leader副本进行处理的。follower副本仅有一个功能，那就是从leader副本拉取消息，尽量让自己跟leader副本的内容一致。
- 给包含重要消息的队列建立一个远端备份。

### 3. Kafka消息存盘不丢消息?

Kafka提供消息持久化，所有数据都会立即被写入文件系统的持久化日志中。kafka一般不会删除消息，不管这些消息有没有被消费。只会根据配置的日志保留时间(log.retention.hours)确认消息多久被删除，默认保留最近一周的日志消息。

### 4. **Kafka消费者不丢失消息**?

- RabbitMQ在消费消息时可以指定是自动应答，还是手动应答。将Kafka的应答模式设定为手动应答可以提高消息消费的可靠性。
- 设置了手动确认，则需要在业务处理成成功后，手动调用`consumer.commitSync()`

**最后，任何用户态的应用程序都无法保证绝对的数据安全，所以，备份与恢复的方案也需要考虑到。**

