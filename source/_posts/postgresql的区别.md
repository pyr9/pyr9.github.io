---
title: postgresql的区别
date: 2022-08-16 16:59:09
tags: 面试题
categories: postgreSQL
---

postgreSQL 和 mysql的对比

- PostgreSQL是进程模式，MySQL是线程模式。进程模式对多CPU利用率比较高。

- 事务的支持：PostgreSQL支持事务的强一致性，事务保证性好，完全支持ACID特性。MySQL只有innodb引擎支持事务，mysiam不支持事务。
- 分区的实现：MySQL分区表的实现要优于PG的基于继承表的分区实现，主要体现在分区个数达到上千上万后的处理性能差异较大。
- PG主表采用堆表存放，MySQL采用索引组织表，能够支持比MySQL更大的数据量。
- MySQL不支持XML 数据类型，PostgreSQL支持
- PostgreSQL完全免费，而且是BSD协议，如果把PostgreSQL改一改，然后再拿去卖钱，也没有人管你，这一点很重要，这表明了PostgreSQL数据库不会被其他公司控制。相反，MySQL现在主要是被Oracle公司控制。
