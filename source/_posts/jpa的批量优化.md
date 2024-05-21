---
title: jpa的批量优化
date: 2024-05-21 19:11:15
tags:
categories:
---

# 1. 问题

Spring-data-jpa自己带的JpaRepository里有很多现有的方法，如findAll, saveAll, deleteAll ,但是saveAll 和deleteAll 这种底层都是foreach的save或者delete的，性能非常低

# 2. 优化

可以通过配置`hibernate.jdbc.batch_size`的方式， 将这些插入操作打包成一个批处理操作，然后一次性发送到数据库执行。

这样做可以减少通信成本和事务开销，提高性能。但依旧不够，所以需要自己写SQL处理器

基于PG：

## 2.1 insert

原理： `insert into %s (%s) values %s;`

注意：

- 需要自己生成ID， 有一张表维护不同业务表的id_value序号，根据当前业务表，查询出当前id_value, 将当前id_value+ LIst.size+1 作为要更新的新id_value, 更新成功的话，则将之前查询出来的id_value+1和最新id_value返回

  `update id_value=`id_value+ LIst.size+1 where id_value=id_value ` 利用CAS

- 获取id失败的话，retry一次

## 2.2 update

原理： `update %s set %s from (values %s) as tmp (%s) where %s.id=tmp.id;`

把fields传过去， 不传默认更新所有

这里就需要更新当前所有的fields， 构建的entity中之遥某个有这个fields，就得都加上



SQL执行可能有长度和行数限制

每次从list里取对象进行SQL拼接时，都检查下当前剩余长度是否够用，以及当前行数是否超过限制

不够的话，再新构建一个SQL将剩下的部分拼接进去

这次的优化速度提高，是倍数增长的，因为之前需要多个SQL，多次连接多次执行，下载都优化成了一次
