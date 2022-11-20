---
title: Explain详解
date: 2021-11-27 13:55:43
tags: Explain详解
categories: 数据库
---

# explain使用介绍

* 使用EXPLAIN关键字可以模拟优化器执行SQL语句，分析你的查询语句或是结构的性能瓶颈 

* 在 select 语句之前增加 explain 关键字，MySQL 会在查询上设置一个标记，执行查询会返回执行计划的信息，而不是执行这条SQL （如果 from 中包含子查询，仍会执行该子查询，将结果放入临时表中）。
* 在查询中的每个表会输出一行，如果有两个表通过 join 连接查询，那么会输出两行。表的意义相当广泛：可以是子查询、一个 union 结果等。

**下面是使用explain的例子：**

```sql 
mysql> explain select * from actor;
```

![](https://tva1.sinaimg.cn/large/008i3skNly1gwztv72dgxj31aq06mq45.jpg)

## explain中的列

### id列

* id列的编号就是select的序列号，有几个select就有几个id

* id的顺序是按select 出现的顺序增长的
* Id列越大执行优先级越高，id相同则从上往下执行，id为NULL最后执行

```sql 
mysql> explain select (select 1 from actor limit 1) from film;
```

![](https://tva1.sinaimg.cn/large/008i3skNly1gwzu3b6nonj31fo07m75y.jpg)

### select_type列

* simple: 简单查询。查询中不含子查询和联合查询。

```sql
mysql> explain select * from film where id = 2;
```

![](https://tva1.sinaimg.cn/large/008i3skNly1gx1xees8p5j31cm06ujso.jpg)

* primary : 复杂查询中最外层的select。

 ```sql
 mysql> explain select (select 1 from actor limit 1) from film;
 ```

中的：`select *** from film ` 

![](https://tva1.sinaimg.cn/large/008i3skNly1gwzu3b6nonj31fo07m75y.jpg)

* subsquery: 复杂查询中的子查询（不在from子句中）。

```sql
mysql> explain select (select 1 from actor limit 1) from film;
```

中的：`select 1 from actor limit 1`

![](https://tva1.sinaimg.cn/large/008i3skNly1gwzu3b6nonj31fo07m75y.jpg)

* derived([dɪˈraɪv]) : 包含在from子句中的子查询。Mysql将把结果存放在一个临时表中，也称为派生表。

```sql 
mysql>  set session optimizer_switch='derived_merge=off';
mysql>  explain select (select 1 from actor where id = 1) from (select * from film where id = 1) der;
```

![](https://tva1.sinaimg.cn/large/008i3skNly1gx1xvhx932j31pa08owgo.jpg)

* union ：在union后的select。

```sql 
mysql> select id,name from actor union all select id, name from film;
```

![](https://tva1.sinaimg.cn/large/008i3skNly1gx399g82jsj31ey07ejsz.jpg)

> *  union 用于把来自多个select  语句的结果组合到一个结果集合中。语法为：
>
>   ```sql 
>  select  column,......from table1
>   union [all]
>  select  column,...... from table2
>   ```

### table列

* 这一列表示explain的这一行在执行哪个表
* 当form后有子查询时，table列为`<derivedN>`格式，表示当前查询依赖id = N的查询，于是先执行id= N的查询。

### type 列

* 这一列表示访问类型，即mysql决定如何查找表中的行。

* 性能排序从好到坏依次为：`system >const> eq_ref> ref> range > index > all`，一般来说，得保证查询至少到range级别，最好到ref。

* 具体的类型介绍。

  > - NULL：mysql在窒息感阶段用不着访问表或者索引。例如索引列中取最小值，可以单独查找索引来完成，不需要执行时访问表。
  >
  >   ```sql 
  >   mysql> explain select min(id) from film;
  >   ```
  >
  >   ![](https://tva1.sinaimg.cn/large/008i3skNly1gx39keqvk8j31kc072myn.jpg)

  > * const：直接按 primary key 或 unique key读取，将该列与常数比较，所以表最多有一个匹配行，读取1次，速度比较快
  >
  >   ```sql 
  >   mysql> explain select id, name from film where id = 1;
  >   ```
  >
  >   ![](https://tva1.sinaimg.cn/large/008i3skNly1gx39u1jkh4j31cm084400.jpg)

  > * eq_ref：primary key 或 unique key 索引的所有部分被连接使用 ，最多只会返回一条符合 条件的记录。这可能是在 const 之外最好的联接类型了，简单的 select 查询不会出现这种 type。
  >
  >   ```sql 
  >   mysql>  explain select * from film_actor left join film on film_actor.film_id = film.id;
  >   ```
  >
  >   ![](https://tva1.sinaimg.cn/large/008i3skNly1gx3a5jmvsej31m207wjt5.jpg)

  > - ref：相比 eq_ref，不使用唯一索引，而是使用普通索引或者联合索引的部分前缀，索引要 和某个值相比较，可能会找到多个符合条件的行。 
  >
  > ```sql 
  > mysql> explain select * from film where name = 'film1';
  > ```
  >
  > ![](https://tva1.sinaimg.cn/large/008i3skNly1gx3a7ju6ugj31fk06mta5.jpg)

  > - range：范围扫描通常出现在 in(), between ,> ,<, >= 等操作中。使用一个索引来检索给定范围的行。 
  >
  > ```sql 
  > mysql> explain select * from actor where id > 1;
  > ```
  >
  > ![](https://tva1.sinaimg.cn/large/008i3skNly1gx3acp599sj31ea06sta0.jpg)

  > - index：扫描全表索引，这通常比ALL快一些。即查询的字段都是索引列。
  >
  > ```sql 
  > mysql>  explain select id,name from film;
  > ```
  >
  > ![](https://tva1.sinaimg.cn/large/008i3skNly1gx3ahaf61zj31eo06mwft.jpg)

  ​		

  > - ALL：即全表扫描，意味着mysql需要从头到尾去查找所需要的行。通常情况下这需要增加索引来进行优化了。
  >
  > ```sql 
  > mysql>  explain select * from actor;
  > ```
  >
  > ![](https://tva1.sinaimg.cn/large/008i3skNly1gx3amcxg5rj31am06yjsm.jpg)

### **possible_keys列** 

* 这一列显示查询可能使用哪些索引来查找。 
* explain 时可能出现 possible_keys 有列，而 key 显示 NULL 的情况，这种情况是因为表中 数据不多，mysql认为索引对此查询帮助不大，选择了全表查询。 
* 如果该列是NULL，则没有相关的索引。在这种情况下，可以通过检查 where 子句看是否可以创造一个适当的索引来提高查询性能，然后用 explain 查看效果。 

###  key列

- 这一列显示mysql实际采用哪个索引来优化对该表的访问。 
- 如果没有使用索引，则该列是 NULL。如果想强制mysql使用或忽视possible_keys列中的索 引，在查询中使用 force index、ignore index。

### key_len列

* 这一列显示了mysql在索引里使用的字节数，通过这个值可以算出具体使用了索引中的哪些列。 
* 举例来说，film_actor的联合索引 idx_film_actor_id 由 film_id 和 actor_id 两个int列组成， 并且每个int是4字节。通过结果中的key_len=4可推断出查询使用了第一个列：film_id列来执行索引查找。 

> key_len计算规则如下： 
>
> * 字符串 
>
>   * char(n)：n字节长度
>
>   * varchar(n)：2字节存储字符串长度，如果是utf-8，则长度 3n 
>
>     \+ 2
>
> * 数值类型
>   * tinyint：1字节 
>   * smallint：2字节
>   * int：4字节 
>   * bigint：8字节
> * 时间类型
>   * date：3字节 
>   * timestamp：4字节 
>   * datetime：8字节
> * 如果字段允许为 NULL，需要1字节记录是否为 NULL

### **ref列** 

- 这一列显示了在key列记录的索引中，表查找值所用到的列或常量，常见的有：const（常量），字段名（例：film.id） 

###  **rows列** 

- 这一列是mysql估计要读取并检测的行数，注意这个不是结果集里的行数。 

### **Extra列**

这一列展示的是额外信息。常见的重要值如下： 

* Using index：使用覆盖索引（覆盖索引指一个查询语句的执行只用从索引中就能够取得，不必从数据表中读取。 ）

* Using where：使用 where 语句来处理结果，查询的列未被索引覆盖

* Using index condition：会先条件过滤索引，过滤完索引后找到所有符合索引条件的数据行，随后用 WHERE 子句中的其他条件去过滤这些数据行；

  ```sql 
  mysql> explain select * from film_actor where film_id > 1;
  ```

* Using temporary：mysql需要创建一张临时表来处理查询。出现这种情况一般是要进行优化的，首先是想到用索引来优化。 

  ```sql 
  mysql> explain select distinct name from actor;
  ```

  -  actor.name没有索引，此时创建了张临时表来distinct 

* Using filesort：将用外部排序而不是索引排序，数据较小时从内存排序，否则需要在磁盘完成排序。这种情况下一般也是要考虑使用索引来优化的。

  ```sql 
  mysql> explain select * from actor order by name;
  ```

  - actor.name未创建索引，会浏览actor整个表，保存排序关键字name和对应的id，然后排 序name并检索行记录 

* Select tables optimized away：使用某些聚合函数（比如 max、min）来访问存在索引的某个字段.

  ```sql 
  mysql> explain select min(id) from film;
  ```



>  Extra 的介绍不是一定的，需要综合当时的场景考虑，不需要记住哪个场景使用的是哪个，只是需要当出现Using filesort, Using temporary：Using where：时，考虑需要优化。







