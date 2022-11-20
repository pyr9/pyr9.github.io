---
title: Kafka 持久化
date: 2022-08-13 17:26:18
tags: 消息队列
categories: Kafka
---

# 1 Kafka 持久化

- 每个 Topic 将消息分成多 Partition，每个 Partition 在存储层面是 append log 文件。
- 任何发布到此 Partition 的消息都会被直接追加到 log 文件的尾部，每条消息在文件中的位置称为 Offest（偏移量）
- Partition 是以文件的形式存储在文件系统中
- log 文件根据 Broker 中的配置保留一定时间后删除来释放磁盘空间。

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h6eje149tgj20ss0hgdgq.jpg" style="zoom:50%;" />
