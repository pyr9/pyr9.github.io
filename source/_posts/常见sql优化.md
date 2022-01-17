---
title: 常见sql优化
date: 2021-12-05 22:08:58
tags: mysql索引
categories: sql优化
---

## 常见sql优化

* 如果索引了多列，要遵守最左前缀法则。指的是查询从索引的最左前列开始并且不跳过索引中的列。
* 尽量使用覆盖索引（只访问索引的查询（索引列包含查询列）），减少select * 语句
* 尽量不使用不等于（！=或者<>），这些无法使用索引，会导致全表扫描 

* 尽量不使用 is null,is not null ，这些无法使用索引，会导致全表扫描 

* like以通配符开头（'$abc...'）mysql索引失效会变成全表扫描操作

* 少用or或in，用它查询时，mysql不一定使用索引，mysql内部优化器会根据检索比例、表大小等多个因素整体评估是否使用索引。

* 尽量将大范围拆分成多个小范围。单次数据量查询过大可能导致优化器最终选择不走索引
* 范围和范围右边的字段不会走索引，只有范围和范围之前的等值字段会走索引

* 不在索引列上做任何操作。（计算、函数、（自动or手动）类型转换），会导致索引失效而转向全表扫描。

准备工作：

```sql
 CREATE TABLE `employees` (
   `id` int(11) NOT NULL AUTO_INCREMENT,
   `name` varchar(24) NOT NULL DEFAULT '' COMMENT '姓名',
   `age` int(11) NOT NULL DEFAULT '0' COMMENT '年龄',
   `position` varchar(20) NOT NULL DEFAULT '' COMMENT '职位',
   `hire_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '入职时 间',
   PRIMARY KEY (`id`),
   KEY `idx_name_age_position` (`name`,`age`,`position`) USING BTREE) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8 COMMENT='员工记录表';
   
   INSERT INTO employees(name,age,position,hire_time) VALUES('LiLei',22,'mana ger',NOW());
   INSERT INTO employees(name,age,position,hire_time) VALUES('HanMeimei', 23,'dev',NOW());
   INSERT INTO employees(name,age,position,hire_time) VALUES('Lucy',23,'dev',NOW());
```

Eg1: 

```sql
mysql> EXPLAIN SELECT * FROM employees WHERE name = 'LiLei';
```

![](https://tva1.sinaimg.cn/large/008i3skNly1gx4g3b8iklj31m606ejsx.jpg)

```sql
mysql> EXPLAIN SELECT * FROM employees WHERE left(name,3) = 'LiLei';
```

![](https://tva1.sinaimg.cn/large/008i3skNly1gx4g3oc8zsj31ek06cgmu.jpg)

Eg2:

```sql
mysql> EXPLAIN select * from employees where date(hire_time) ='2018-09-30';
```

```sql
mysql> EXPLAIN select * from employees where hire_time >='2018-09-30 00:00:00' and hire_time <='2018-09-30 23:59:59';
```

给hire_time增加一个普通索引，上面的sql由于进行了date运算所以不会走索引，下面的会走。



## **Using filesort文件排序原理详解** 

* MySQL支持两种方式的排序filesort和index，Using index是指MySQL扫描索引本身完成排序。index效率高，filesort效率低。 

* filesort文件排序方式

  * 单路排序：是一次性取出满足条件行的所有字段，然后在sort buffer中进行排序；用trace工具可以看到sort_mode信息里显示< sort_key, additional_fields >或者< sort_key, packed_additional_fields >
  * 双路排序（又叫**回表**排序模式）：是首先根据相应的条件取出相应的**排序字段**和**可以直接定位行** **数据的行 ID**(主键)，然后在 sort buffer 中进行排序，排序完后需要再次通过主键回到原表查询需要的字段。用trace工具 可以看到sort_mode信息里显示< sort_key, rowid > 

  >  MySQL 通过比较系统变量 max_length_for_sort_data(**默认1024字节**) 的大小和需要查询的字段总大小来 判断使用哪种排序模式。
  >
  > *  如果 max_length_for_sort_data 比查询字段的总长度大，那么使用 单路排序模式； 
  >
  > *  如果 max_length_for_sort_data 比查询字段的总长度小，那么使用 双路排序模式。



* 单路排序 和 双路排序的选择

  * 如果 MySQL 排序内存配置的比较小并且没有条件继续增加了，可以适当max_length_for_sort_data 配 置小点，让优化器选择使用**双路排序**算法，可以在sort_buffer 中一次排序更多的行，只是需要再根据主键回到原表取数据。 

  * 如果 MySQL 排序内存有条件可以配置比较大，可以适当增大 max_length_for_sort_data 的值，让优化器 优先选择全字段排序(**单路排序**)，把需要的字段放到 sort_buffer 中，这样排序后就会直接从内存里返回查 询结果了。 

> 所以，MySQL通过 **max_length_for_sort_data** 这个参数来控制排序，在不同场景使用不同的排序模式， 从而提升排序效率。
