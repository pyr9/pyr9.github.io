---
title: Redis主从架构
date: 2021-12-28 22:45:20
tags: redis主从架构
categories: redis
---

# Redis主从架构介绍

单机的 redis，能够承载的 QPS 大概就在上万到几万不等。对于缓存来说，一般都是用来支撑读高并发的。因此架构做成主从(master-slave)架构，**一主多从，主负责写，并且将数据复制到其它的 slave 节点，从节点负责读**。所有的读请求全部走从节点。这样也可以很轻松实现水平扩容，支撑**读高并发**。



<img src="https://tva1.sinaimg.cn/large/008i3skNly1gxv2a0o0mqj30oi0lsgmm.jpg" style="zoom:30%;" />

# 主从复制(全量复制)流程

![](https://tva1.sinaimg.cn/large/008i3skNly1gxv2yicirhj30tq0lggnx.jpg)

1. 如果你为master配置了一个slave，不管这个slave是否是第一次连接上Master，它都会发送一个**SYNC**命令(redis2.8版本之前的命令)master请求复制数据。
2. master收到SYNC命令后，会在后台进行数据持久化通过bgsave生成最新的rdb快照文件。
3. master将生成的rdb数据发给slave。
4. Master由于从执行bgsave到生成rdb数据存在时间，在此时间间隔内，可能有新的客户端请求，master会把这些最近的请求缓存在内存中。（也就是一个最近数据的缓冲区，默认大小为1M）
5. 当第3步的rdb数据发完后，他会把这些缓存的数据也发给slave
6. 当slave有老的rdb数据，就会先把老的数据清掉。重新生成包含缓存的rd b数据，并加载到内存中。
7. 之后，master和slave通过socket长连接，持续进行命令同步，从而保证主从数据一致。

> 当master与slave之间的连接由于某些原因而断开时，slave能够自动重连Master。

> 如果master收到了多个slave并发连接请求，它只会进行一次持久化，而不是一个连接一次，然后再把这一份持久化的数据发送给多个并发连接的slave。  

**当master和slave断开重连后，一般都会对整份数据进行复制。但从redis2.8版本开始，master和slave断开重连后支持部分复制**。

# 主从复制(部分复制)流程

![](https://tva1.sinaimg.cn/large/008i3skNly1gxv2xkkaizj318w0ro78b.jpg)

1. 从2.8版本开始，slave与master能够在网络连接断开**重连后只进行部分数据复制。** 
2. master会在其内存中创建一个复制数据用的缓存队列，缓存最近一段时间的数据。master和它所有的slave都维护了复制的数据下标offset和master的进程id。
3. 当网络连接断开后，slave会请求master继续进行未完成的复制。
   - 从节点数据下标 offset还在缓存队列里，那么将会从slave记录的数据下标开始从缓存区复制。
   - 如果master进程id变化了，或者从节点数据下标 offset太旧，已经不在master的缓存队列里了，那么将会进行一次全量数据的复制

# **Redis哨兵高可用架构** 

- sentinel哨兵是特殊的redis服务，不提供读写服务，主要用来监控redis实例节点。 

- 哨兵架构下client端第一次从哨兵找出redis的主节点，后续就直接访问redis的主节点，不会每次都通过sentinel代理访问redis的主节点。
- 当redis的主节点发生变化，哨兵会第一时间感知到，并且将新的redis主节点通知给client端(这里面redis的client端一般都实现了订阅功能，订阅sentinel发布的节点变动消息) 

![](https://tva1.sinaimg.cn/large/008i3skNly1gxv33qmtvpj319e0pktbv.jpg)

