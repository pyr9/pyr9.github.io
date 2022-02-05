---
title: ElasticSearch简介
date: 2021-09-28 20:10:27
tags: 分布式框架
categories: ElasticSearch
---

# ElasticSearch简介

## Es是什么？

- Elasticsearch是用Java开发并且是当前最流行的开源的企业级搜索引擎。
- 能够达到实时搜索，稳定，可靠，快速，安装使用方便。
- 客户端支持Java、.NET（C#）、PHP、Python、Ruby等多种语言。

## 应用场景

关键词搜索。

![](https://tva1.sinaimg.cn/large/008i3skNly1gz2w2u8asuj30vt0u044p.jpg)

> 从上面的搜索结果，可以看出不是根据模糊匹配去进行搜索的

## ElasticSearch与Lucene的关系

- Lucene可以被认为是迄今为止最先进、性能最好的、功能最全的搜索引擎库（框架）

### Lucene缺点：

- 想要使用Lucene，必须使用Java来作为开发语言并将其直接集成到你的应用中。
- Lucene的配置及使用非常复杂，你需要深入了解检索的相关知识来理解它是如何工作的。
- 使用非常复杂-创建索引和搜索索引代码繁杂。
- 不支持集群环境-索引数据不同步（不支持大型项目）
- 索引数据如果太多就不行，索引库和应用所在同一个服务器,共同占用硬盘.共用空间少.

**上述Lucene框架中的缺点,ES全部都能解决.**

## 哪些公司在使用Elasticsearch

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gz2vhwoyi1j309q0isq3h.jpg" style="zoom:50%;" />



## ES vs Solr比较

- 当单纯的对已有数据进行搜索时，Solr更快。
- 当实时建立索引时,Solr会产生io阻塞，查询性能较差,Elasticsearch具有明显的优势。
- Solr利用Zookeeper进行分布式管理，而Elasticsearch自身带有分布式协调管理功能。
- Solr支持更多格式的数据，比如JSON、XML、CSV，而Elasticsearch仅支持json文件格式。
- Solr在传统的搜索应用中表现好于Elasticsearch，但在处理实时搜索应用时效率明显低于Elasticsearch。
- Solr是传统搜索应用的有力解决方案，但Elasticsearch更适用于新兴的实时搜索应用。



## ESvs关系型数据库

![](https://tva1.sinaimg.cn/large/008i3skNly1gz2vkt8j0jj30zu0fsgn8.jpg)

## Lucene全文检索框架

### 什么是全文检索

- 全文检索是指：通过一个程序扫描文本中的每一个单词，针对单词建立索引，并保存该单词在文本中的位置、以及出现的次数。
- 用户查询时，通过之前建立好的索引来查询，将索引中单词对应的文本位置、出现的次数返回给用户，因为有了具体文本的位置，所以就可以将具体内容读取出来了

### 词原理之倒排索引

对于关系型数据库mysql来说，**普通的索引结构就是“id->题目->内容”，**在我们搜索的时候，如果我们知道id或者题目**，那么检索效率是很高效的，因为“id”、“题目”是很方便创建索引的。**

那么**倒排序索引**的结构是怎样的呢？简单来讲**就是“以内容的关键词”建立索引，**映射关系为**“内容的关键词->ID”。**这样的话，我们只需要在“关键词”中进行检索

![](https://tva1.sinaimg.cn/large/008i3skNly1gz2vpv36j1j30z40em40a.jpg)

> 对于上面的场景，输入为hello，会根据hello定位到此时数据的id为1和2，进而查出对应的数据。

## Elasticsearch中的核心概念

### 索引index

- 一个索引就是一个拥有几分相似特征的文档的集合。比如说，可以有一个客户数据的索引，另一个产品目录的索引，还有一个订单数据的索引
- 一个索引由一个名字来标识（必须全部是小写字母的），并且当我们要对对应于这个索引中的文档进行索引、搜索、更新和删除的时候，都要使用到这个名字

### 映射mapping

- ElasticSearch中的映射（Mapping）用来定义一个文档

- mapping是处理数据的方式和规则方面做一些限制，如某个字段的数据类型、默认值、分词器、是否被索引等等，这些都是映射里面可以设置的

### 字段Field

- 相当于是数据表的字段|列

### 字段类型Type

每一个字段都应该有一个对应的类型，例如：Text、Keyword、Byte等

### 文档document

一个文档是一个可被索引的基础信息单元，类似一条记录。文档以JSON（JavascriptObjectNotation）格式来表示；



### 

