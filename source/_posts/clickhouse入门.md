---
title: clickhouse入门
date: 2022-02-01 14:27:15
tags: 分布式框架
categories: clickhouse
---

# **ClickHouse** 是什么？

ClickHouse 是俄罗斯的 Yandex 于 2016 年开源的列式存储数据库（DBMS），使用 C++

语言编写，主要用于在线分析处理查询（OLAP），能够使用 SQL 查询实时生成分析数据报

告。

>- OLTP:   联机事务处理（Online Transaction Processing)。是mysql，oracle这种传统的关系型数据库的主要应用，适合做基本的日常的事务处理，增删改查等操作。
>- OLAP：联机分析处理（Online Analytical Processing)，支持复杂的分析操作，侧重决策支持，并且提供直观易懂的查询结果。适合做一次插入多次查询，适合分析，聚合的场景，但是对更新，删除不擅长。



# ClickHouse **的特点**

## 1.**列式存储**

### 行式存储和列式存储的对比

- 行式存储的好处是想查某个人所有的属性时，可以通过一次磁盘查找加顺序读取就可以。但是当想查所有人的年龄时，需要不停的查找，或者全表扫描才行，遍历的很多数据都是不需要的。

- 列式存储，想查所有人的年龄只需把年龄那一列拿出来就可以了

以下面的表为例：

![](https://tva1.sinaimg.cn/large/008i3skNly1gyy3x95nv3j31580est9i.jpg)

- 采用行式存储时，数据在磁盘上的组织结构为：

  ![](https://tva1.sinaimg.cn/large/008i3skNly1gyy3xu1e5zj315s05igm7.jpg)

- 采用列式存储时，数据在磁盘上的组织结构为：

  ![](https://tva1.sinaimg.cn/large/008i3skNly1gyy3yqqgzzj315405kjrz.jpg)

### **列式储存的好处：**

- 对于列的聚合，计数，求和等统计操作原因优于行式存储。

- 由于某一列的数据类型都是相同的，针对于数据存储更容易进行数据压缩，每一列

  选择更优的数据压缩算法，大大提高了数据的压缩比重。

- 由于数据压缩比更好，一方面节省了磁盘空间，另一方面对于 cache 也有了更大的

发挥空间

## **DBMS** **的功能**

几乎覆盖了标准 SQL 的大部分语法，包括 DDL 和 DML，以及配套的各种函数，用户管

理及权限管理，数据的备份与恢复。 

> 数据库管理系统（英语：database management system，缩写：DBMS）即数据库管理软件，是一种针对对象数据库，为管理数据库而设计的大型计算机软件管理系统。
>
> 具有代表性的数据管理系统有：Oracle、Microsoft SQL Server、Access、MySQL及PostgreSQL等。通常数据库管理师会使用数据库管理系统来创建数据库系统。

##  **多样化引擎**

ClickHouse 和 MySQL 类似，把表级的存储引擎插件化，根据表的不同需求可以设定不同

的存储引擎。目前包括合并树、日志、接口和其他四大类 20 多种引擎。

## 高吞吐写入能力

- ClickHouse 采用类 LSM Tree的结构，数据写入后定期在后台 Compaction。通过类 LSM tree的结构，ClickHouse 在数据导入时全部是顺序 append 写，写入后数据段不可更改，在后台compaction 时也是多个段 merge sort 后顺序写回磁盘。顺序写的特性，充分利用了磁盘的吞吐能力，即便在 HDD 上也有着优异的写入性能。

- 官方公开 benchmark 测试显示能够达到 50MB-200MB/s 的写入吞吐能力，按照每行

100Byte 估算，大约相当于 50W-200W 条/s 的写入速度。

[深入浅出分析LSM树（日志结构合并树） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/415799237)

> ![](https://tva1.sinaimg.cn/large/008i3skNly1gyy4aob591j314o0hg0uk.jpg)

### **数据分区与线程级并行**

- ClickHouse 将数据划分为多个 partition，每个 partition 再进一步划分为多个 index 

granularity(索引粒度)，然后通过多个 CPU核心分别处理其中的一部分来实现并行数据处理。

在这种设计下，**单条 Query 就能利用整机所有 CPU**。极致的并行处理能力，极大的降低了查

询延时。

- 所以，ClickHouse 即使对于大量数据的查询也能够化整为零平行处理。但是有一个弊端

就是对于单条查询使用多 cpu，就不利于同时并发多条查询。所以对于高 qps 的查询业务，

ClickHouse 并不是强项。

> QPS（**Query Per Second**）：每秒请求数，就是说服务器在一秒的时间内处理了多少个请求。

