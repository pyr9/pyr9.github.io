---
title: ElasticSearch乐观并发控制
date: 2022-03-07 21:59:29
tags: 分布式框架
categories: ElasticSearch
---

# Elasticsearch乐观并发控制

Elasticsearch 中使用的这种乐观的方式假定冲突是不可能发生的，并且不会阻塞正在 尝试的操作。 然而，如果源数据在读写当中被修改，更新将会失败。应用程序接下来将决定该如何解决冲突。 例如，可以重试更新、使用新的数据、或者将相关情况报告给用户。 

> 在数据库领域中，有两种方法来确保并发更新，不会丢失数据： 分别为乐观和悲观

## 举例

以创建一个文档为例 

step1: 创建文档

```java
POST /test1/_doc/1
{
  "Id": 1,
  "name": "张三丰"
}
```

```java
{
  "_index" : "test1",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

**使用if_seq_no=版本值 &if_primary_term=文档位置进行并发版本控制**

- _seq_no：文档版本号，**version属于当个文档，而seq_no属于整个index**

- _primary_term：_primary_term也和_seq_no一样都是整数，每当Primary Shard发生重新分配时，比如重启，Primary选举等，_primary_term会递增1

step2: 更新文档

```java
POST /test1/_update/1/?if_seq_no=0&if_primary_term=1
{
  "doc": {
    "name": "张三丰666"
  }
}
```

```java
{
  "_index" : "test1",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
```

## 
