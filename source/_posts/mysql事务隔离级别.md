---
title: mysql事务隔离级别
date: 2022-06-14 22:28:21
tags: mysql事务
categories: mysql
---

# mysql 事务

## 1.事务定义

事务是一个操作集合，这些操作要么都执行，要么都不执行，他是一个不可分割的工作单位。

## 2.事务的四大特性

- 原子性：事务是一个原子操作单元，它对数据的修改，要么都执行，要么都不执行。
- 一致性：一个事务执行前和执行后，数据必须保持一致，如：转账前用户AB的钱加在一起时500，转账后也应该是500
- 隔离型：事务外的实体不可以知道事务过程中的中间状态
- 持久性：对数据库的操作是永久性的，即使系统故障也能保持

## 3.不考虑事务的隔离性会产生并发问题

- 更新丢失：当两个或者多个事务同时对一行数据进行更新，会发生数据的覆盖，最后的更新覆盖了其他事务的更新。
- 脏读：读到了没有提交的数据，一个事务正在写操作，另一个事务进行了读操作，读到了脏数据。如果此时事务回滚，读取到的数据就是无效的。
- 不可重复读：读到了已经提交的数据，事务A多次读取同一数据，但在这个过程中，事务B对数据进行了修改并提交，会导致事务A多次读取数据结果不一致。
- 幻读：事务A读取到了事务B提交的新增数据。查询某记录是否存在，不存在，准备插入此记录，但执行 insert 时发现此记录已存在，无法插入，此时就发生了幻读。

# mysql 事务隔离级别

- 脏读”、“不可重复读”和“幻读”,其实都是数据库读一致性问题,必须由数据库提供一定的事务隔离机制来解决。

- mysql隔离级别越高，性能越差。因为事务的本质就是串行化，这显然与并发是矛盾的。
- `RR` 级别作为 `mysql` 事务默认隔离级别，是事务安全与性能的折中。
- 查看当前事务的隔离级别 `select @@transaction_isolation;`
- 设置事务隔离级别
  - 当前会话： `SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;`
  - 全局：`SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;`

| 隔离级别 | 脏读   | 不可重复读 |  幻读  |
| -------- | ------ | ---------- | :----: |
| 读未提交 | 可能   | 可能       |  可能  |
| 读已提交 | 不可能 | 可能       |  可能  |
| 可重复读 | 不可能 | 不可能     |  可能  |
| 串行化   | 不可能 | 不可能     | 不可能 |

## 1.读未提交

- 打开两个客户端：客户端A设置当前事务模式为read uncommitted，查询employees 表的初始值

  ```sql
  mysql> select @@transaction_isolation;
  +-------------------------+
  | @@transaction_isolation |
  +-------------------------+
  | READ-UNCOMMITTED        |
  +-------------------------+
  1 row in set (0.00 sec)
  
  mysql>  select * from employees;
  +----+-----------+-----+----------+---------------------+
  | id | name      | age | position | hire_time           |
  +----+-----------+-----+----------+---------------------+
  |  4 | LiLei     |  22 | mana ger | 2021-12-06 21:36:50 |
  |  5 | HanMeimei |  23 | dev      | 2021-12-06 21:36:50 |
  |  6 | Lucy      |  23 | dev      | 2021-12-06 21:36:50 |
  +----+-----------+-----+----------+---------------------+
  3 rows in set (0.00 sec)
  ```

- 客户端B开启事务，并执行更新操作

  ```sql
  mysql> begin;
  Query OK, 0 rows affected (0.00 sec)
  
  mysql>  update employees set name = "newLiLei" where id = 4;
  Query OK, 1 row affected (0.00 sec)
  Rows matched: 1  Changed: 1  Warnings: 0
  ```

- 这时，虽然客户端B的事务还没提交，但是客户端A就可以查询到B已经更新的数据，一旦客户端B的事务因为某种原因回滚，所有的操作都将会被撤销，那客户端A查询到的数据其实就是**脏数据**： 

- ```sql
  mysql>  select * from employees;
  +----+-----------+-----+----------+---------------------+
  | id | name      | age | position | hire_time           |
  +----+-----------+-----+----------+---------------------+
  |  4 | newLiLei  |  22 | mana ger | 2021-12-06 21:36:50 |
  |  5 | HanMeimei |  23 | dev      | 2021-12-06 21:36:50 |
  |  6 | Lucy      |  23 | dev      | 2021-12-06 21:36:50 |
  +----+-----------+-----+----------+---------------------+
  3 rows in set (0.00 sec)
  ```

## 读已提交

- 打开两个客户端：客户端A设置当前事务模式为read committed，查询employees 表的初始值

```sql
mysql> select @@transaction_isolation;
+-------------------------+
| @@transaction_isolation |
+-------------------------+
| READ-COMMITTED          |
+-------------------------+
1 row in set (0.00 sec)

mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from employees;
+----+-----------+-----+----------+---------------------+
| id | name      | age | position | hire_time           |
+----+-----------+-----+----------+---------------------+
|  4 | LiLei     |  22 | mana ger | 2021-12-06 21:36:50 |
|  5 | HanMeimei |  23 | dev      | 2021-12-06 21:36:50 |
|  6 | Lucy      |  23 | dev      | 2021-12-06 21:36:50 |
+----+-----------+-----+----------+---------------------+
3 rows in set (0.00 sec)
```

- 客户端B开启事务，并执行更新操作

```sql
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql>  update employees set name = "newLiLei" where id = 4;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

- 这时，客户端B的事务还没提交，客户端A不能查询到B已经更新的数据，解决了脏读问题：

```sql
mysql> select * from employees;
+----+-----------+-----+----------+---------------------+
| id | name      | age | position | hire_time           |
+----+-----------+-----+----------+---------------------+
|  4 | LiLei     |  22 | mana ger | 2021-12-06 21:36:50 |
|  5 | HanMeimei |  23 | dev      | 2021-12-06 21:36:50 |
|  6 | Lucy      |  23 | dev      | 2021-12-06 21:36:50 |
+----+-----------+-----+----------+---------------------+
3 rows in set (0.00 sec)
```

- 客户端B的事务提交

```sql
mysql> commit;
Query OK, 0 rows affected (0.01 sec)
```

- 客户端A再次查询，结果 与上一步不一致，即产生了不可重复读的问题

```sql
mysql> select * from employees;
+----+-----------+-----+----------+---------------------+
| id | name      | age | position | hire_time           |
+----+-----------+-----+----------+---------------------+
|  4 | newLiLei  |  22 | mana ger | 2021-12-06 21:36:50 |
|  5 | HanMeimei |  23 | dev      | 2021-12-06 21:36:50 |
|  6 | Lucy      |  23 | dev      | 2021-12-06 21:36:50 |
+----+-----------+-----+----------+---------------------+
```



## 可重复读

- 打开两个客户端：客户端A设置当前事务模式为REPEATABLE-READ，查询employees 表的初始值

```sql
mysql> select @@transaction_isolation;
+-------------------------+
| @@transaction_isolation |
+-------------------------+
| REPEATABLE-READ         |
+-------------------------+
1 row in set (0.00 sec)

mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from employees;
+----+-----------+-----+----------+---------------------+
| id | name      | age | position | hire_time           |
+----+-----------+-----+----------+---------------------+
|  4 | LiLei     |  22 | mana ger | 2021-12-06 21:36:50 |
|  5 | HanMeimei |  23 | dev      | 2021-12-06 21:36:50 |
|  6 | Lucy      |  23 | dev      | 2021-12-06 21:36:50 |
+----+-----------+-----+----------+---------------------+
3 rows in set (0.00 sec)
```

- 客户端B开启事务，并执行更新操作, 并提交。

```sql
mysql> begin;
Query OK, 0 rows affected (0.00 sec)

mysql>  update employees set name = "newLiLei" where id = 4;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

- 在客户端A查询表account的所有记录，与上一次查询结果一致，没出现有出现不可重复读的问题

```sql
mysql> select * from employees;
+----+-----------+-----+----------+---------------------+
| id | name      | age | position | hire_time           |
+----+-----------+-----+----------+---------------------+
|  4 | LiLei     |  22 | mana ger | 2021-12-06 21:36:50 |
|  5 | HanMeimei |  23 | dev      | 2021-12-06 21:36:50 |
|  6 | Lucy      |  23 | dev      | 2021-12-06 21:36:50 |
+----+-----------+-----+----------+---------------------+
3 rows in set (0.01 sec)
```

> Mysql默认级别是repeatable-read，有办法解决幻读问题吗？ 
>
> **间隙锁在某些情况下可以解决幻读问题** 
>
> 要避免幻读可以用间隙锁在Session_1下面执行update account set name = 'zhuge' where id > 10 and id <=20;，则其他Session没法在这个范围所包含的间隙里插入或修改任何数据

## 串行化

- 打开两个客户端：客户端A设置当前事务模式为SERIALIZABLE，查询employees 表的初始值

```sql
mysql> select @@transaction_isolation;
+-------------------------+
| @@transaction_isolation |
+-------------------------+
| SERIALIZABLE            |
+-------------------------+
1 row in set (0.00 sec) 

mysql> select * from employees;
+----+-----------+-----+----------+---------------------+
| id | name      | age | position | hire_time           |
+----+-----------+-----+----------+---------------------+
|  4 | newLiLei  |  22 | mana ger | 2021-12-06 21:36:50 |
|  5 | HanMeimei |  23 | dev      | 2021-12-06 21:36:50 |
|  6 | Lucy123   |  23 | dev      | 2021-12-06 21:36:50 |
|  7 | wangwu123 |  18 | super    | 2021-12-06 21:36:53 |
+----+-----------+-----+----------+---------------------+
```

- 打开一个客户端B，并设置当前事务模式为serializable，插入一条记录报错，表被锁了插入失败，mysql中事务隔离级别为serializable时会锁表，因此不会出现幻读的情况，这种隔离级别并发性极低，开发中很少会用到。 

```sql
mysql> begin;
Query OK, 0 rows affected (0.00 sec)
mysql> insert into employees values(12,"lili2",11,"manager","2021-12-06 21:36:55");
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
```





# InnoDB与MYISAM的最大不同有两点：

1. 支持事务
2. 支持行锁



# 性能优化建议

- 尽可能低级别事务隔离

- 尽可能减少检索条件范围，避免间隙锁 

- 尽量控制事务大小，减少锁定资源量和时间长度，涉及事务加锁的sql尽量放在事务最后执行 

- 尽可能让所有数据检索都通过索引来完成，避免无索引行锁升级为表锁 

  > 无索引行锁会升级为表锁：锁主要是加在索引上，如果对非索引字段更新, 行锁可能会变表锁 
