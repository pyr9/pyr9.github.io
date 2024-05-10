---
title: Mongodb集群
date: 2022-09-22 23:01:54
tags: 集群
categories: MongoDB
---

# 1. 主从模式

主从最大的问题就是无法自动故障转移，对于简单的主从复制无法自动故障转移的缺陷，各个数据库都在改进，MySQL推出的Fabric ，Redis的哨兵，Mongodb的复制集。

# 2. 复制集模式

- Mongodb复制集（Replication Set）由一组Mongod实例（进程）组成，包含一个 Primary节点和多个Secondary节点
- Mongodb Driver（客户端）的所有数据都写入 ，Primary，Secondary从Primary同步写入的数据，以保持复制集内所有成员存储相同的数据集，提供数据的高可用
- 从节点会从oplog拉数据，类似mysql的binlog
- 在实现高可用的同时，复制集实现了其他几个附加作用: 
  - 数据分发: 将数据从一个区域复制到另一个区域，减少另一个区域的读延迟 
  - 读写分离: 不同类型的压力分别在不同的节点上执行 
  - 异地容灾: 在数据中心故障时候快速切换到异地 

![image-20230228224723229](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228224723229.png)

## 2.1 **三节点复制集模式**

常见的复制集架构由3个成员节点组成，其中存在几种不同的模式

### 2.1.1 **PSS模式（官方推荐模式）** 

- PSS模式由一个主节点和两个备节点所组成，即Primary+Secondary+Secondary

- 此模式始终提供数据集的两个完整副本，如果主节点不可用，则复制集选择备节点作为主节 点并继续正常操作。
- 旧的主节点在可用时重新加入复制集。 

![image-20230228224732217](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228224732217.png)

### 2.1.2 ****PSA**模式（官方推荐模式）**

- PSA模式由一个主节点、一个备节点和一个仲裁者节点组成，即 Primary+Secondary+Arbiter 
- 此模式仅提供数据的一个完整副本，如果主节点不可用，则复制 集将选择备节点作为主节点。
- Arbiter节点不存储数据副本，也不提供业务的读写操作。Arbiter节点发生故障不影 响业务，仅影响选举投票。因为需要半数选举，但机器成本不够再申请一台

![image-20230228224740040](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228224740040.png)

# 3 **分片集群架构** 

MongoDB 分片集群（Sharded Cluster）是对数据进行水平扩展的一种方式

## 3.1 使用场景

- 写IOPS超出单个MongoDB节点的写服务能力
- 存储容量需求超出单机的磁盘容量。 
- 活跃的数据集超出单机内存容量，导致很多请求都要从磁盘读取数据，影响性 能
