---
title: ElasticSearch集群架构
date: 2022-09-22 21:26:48
tags: 集群
categories: ElasticSearch
---

# 1 **ES集群架构**

- 一个集群可以有一个或者多个节点 

- 不同的集群通过不同的名字来区分，默认名字“elasticsearch“ 

- 通过配置文件修改，或者在命令行中 -E cluster.name=es-cluster进行设定

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h6fpfb23tij21ay0men14.jpg" style="zoom:50%;" />

# 2  集群节点类型

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h6fp2uzs1oj21ac0h6mz9.jpg" style="zoom:50%;" />

- Master Node：主节点 

  - 处理创建，删除索引等请求，负责索引的创建与删除

  - 决定分片被分配到哪个节点

  - 管理集群节点状态

- Master eligible nodes：

  - 可以参与选举的合格节点 
  - 每个节点启动后，默认就是一个Master eligible节点 

- Data Node：数据节点 

  - 可以保存数据的节点，负责数据写入与搜索
  - 节点启动后，默认就是数据节点。可以设置node.data: false 禁止

- Coordinating Node：协调节点

  - 负责接受Client的请求， 将请求分发到合适的节点，最终把结果汇集到一起 
  - 每个节点默认都起到了Coordinating Node的职责 

# 3 分片和副本

## 3.1 分片

- ES 通过水平拆分的方式将一个索引上的数据拆分出来分配到不同的数据块上，分布在多台服务器上存储，拆分出来的数据库块称之为一个分片。

- 一个索引（index）由多个shard（分片）组成，而分片是分布在不同的服务器上的

- 主分片数在索引创建时指定，后续不允许修改，除非Reindex 

  

## 3.2 副本

- 用以解决数据高可用的问题。 副本分片是主分片的拷贝 
- 副本分片数，可以动态调整 
- 增加副本数，还可以在一定程度上提高服务的可用性(读取的吞 吐) 

```java
1 # 指定索引的主分片和副本分片数 
  PUT /blogs  {
 "settings": {
   "number_of_shards": 3,
   "number_of_replicas": 1 
 } }
```

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h6fpbrd4qkj21au0maq4v.jpg" style="zoom:50%;" />

## 3.2.1 **设置副本分片数** 

副本是主分片的拷贝： 

- 提高系统可用性︰响应查询请求，防止数据丢失 

- 需要占用和主分片一样的资源 

对性能的影响：

- 分片间的数据同步会占用一定的网络带宽，影响效率

- 会减缓对主分片的查询压力，但是会消耗同样的内存资源。如果机器资源充分， 提高副本数，可以提高整体的查询QPS

# 4 Elasticsearch文档写入原理

1. 客户端选择一个node发送请求过去，这个node就是coordinating node (协调节 点)
2. coordinating node计算得到文档要写入的分片。默认是 `hash(文档的id) % 主分片数`
3. coordinating node，对document进行路由，将请求转发给对应的node 
4. node上的primary shard处理请求，然后将数据同步到replica node 
5. coordinating node如果发现primary node和所有的replica node都保存好了文档， 就会返回请求到客户端 

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h6fpsiccuqj21ds0nwmzn.jpg)

# 5 **ES读取数据的过程** 

## 5. 1 **根据id查询数据的过程**

1. 客户端发送请求到任意一个 node，成为 coordinate node
2. coordinate node 对 doc id 进行哈希路由，将请求转发到对应的 node，此时会 使用 round-robin 随机轮询算法，在 primary shard 以及其所有 replica 中随机选择一个，让读请求负载均衡。
3.  接收请求的 node 返回 document 给 coordinate node
4. coordinate node 返回 document 给客户端。 

> 根据 doc id 进行 hash，判断出来当时把 doc id 分配到了哪个 shard 上面去，从那个 shard 去查询。

## 5.2 **根据关键词查询数据的过程** 

1. 客户端发送请求到一个 coordinate node 。
2. 协调节点将查询请求广播到每一个数据节点，这些数据节点的分片会处理该查询请求 
3. **query phase**：每个 shard 将自己的搜索结果（包括：文档ID、节点信息、分片信息）返回给协调节点，由协调节点进行 数据的合并、排序、分页等操作，产出最终结果
4. **fetch phase：**接着由协调节点根据 doc id 去各个节点上拉取实际的 document 数据，最终返回给客户端。
