---
title: kafka设计原理
date: 2022-08-13 17:26:18
tags: 消息队列
categories: Kafka
---

# Kafka 设计原理

## 消费者rebalance机制

#### 服务端怎么确认触发rebalance？

通过心跳，设置一个心跳时间，要是指定时间内没有响应，就通过心跳，下发rebalance命令到其他consumer，通知他们进行rebalance
