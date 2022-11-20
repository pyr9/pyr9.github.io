---
title: clickhouse表引擎
date: 2022-02-03 13:24:24
tags: 数据库
categories: Clickhouse
---

# 表引擎介绍

表引擎是 ClickHouse 的一大特色。不同的引擎有不同的作用，可以说， 表引擎决定了如何存储表的数据。包括：

➢ 数据的存储方式和位置，写到哪里以及从哪里读取数据。 （一般的引擎都存在磁盘中，但是存在与其他数据库继承的场景，比如和mysql集成，数据放在mysql服务端。）

➢ 支持哪些查询以及如何支持。（有些语法只有在特定的引擎才可以使用，比如不能在 MergeTree 表中存储多维数组）

➢ 并发数据访问。（可以多线程执行同一条sql）

➢ 索引的使用（如果存在）。

➢ 是否可以执行多线程请求。

➢ 数据复制参数。

> 表引擎的使用方式就是必须显式在创建表时定义该表使用的引擎，以及引擎使用的相关参数。

**特别注意：引擎的名称大小写敏感**

# 合并树家族

##  **MergeTree**

- ClickHouse 中最强大的表引擎当属 MergeTree（合并树）引擎及该系列（*MergeTree）

中的其他引擎，支持索引和分区，地位可以相当于 innodb 之于 Mysql。而且基于 MergeTree，

- 还衍生除了很多小弟，也是非常有特色的引擎。

1. 建表语句

   ```sql
   create table t_order_mt(
   id UInt32,
   sku_id String,
   total_amount Decimal(16,2),
   create_time Datetime
   ) engine =MergeTree
   partition by toYYYYMMDD(create_time)
   primary key (id)
   order by (id,sku_id);
   ```

2. **插入数据**

```sql
insert into t_order_mt values
(101,'sku_001',1000.00,'2020-06-01 12:00:00') ,
(102,'sku_002',2000.00,'2020-06-01 11:00:00'),
(102,'sku_004',2500.00,'2020-06-01 12:00:00'),
(102,'sku_002',600.00,'2020-06-02 12:00:00');
```

### 参数介绍

#### **partition by** 分区

##### 1. 目的

分区的目的主要是降低扫描的范围，优化查询速度

##### 2. 可选

可选的，如果不填，只会使用一个分区。

##### 3.**分区目录**

MergeTree 是以列文件+索引文件+表定义文件组成的，但是如果设定了分区那么这些文

件就会保存到不同的分区目录中。

![](https://tva1.sinaimg.cn/large/008i3skNly1gz0c4bfsguj30rw0iqjtg.jpg)

##### 4.**并行**

分区后，面对涉及跨分区的查询统计，ClickHouse 会以分区为单位并行处理。

##### 5.**数据写入与分区合并**

- 任何一个批次的数据写入都会产生一个临时分区，不会纳入任何一个已有的分区。

- 写入后的某个时刻（大概 10-15 分钟后），ClickHouse 会自动执行合并操作。

- 也可以手动通过 optimize 执行，把临时分区的数据，合并到已有分区中。语法：

  ```sql
  optimize table xxxx final;
  ```

  如：

  - 再次执行上面的插入操作

  ```sql
  insert into t_order_mt values
  (101,'sku_001',1000.00,'2020-06-01 12:00:00') ,
  (102,'sku_002',2000.00,'2020-06-01 11:00:00'),
  (102,'sku_004',2500.00,'2020-06-01 12:00:00'),
  (102,'sku_002',600.00,'2020-06-02 12:00:00');
  ```

  - 查看数据并没有纳入任何分区

  ![](https://tva1.sinaimg.cn/large/008i3skNly1gz0c7984cnj30ui0s8wi5.jpg)

- 手动 optimize 之后，再次查询  ![](https://tva1.sinaimg.cn/large/008i3skNly1gz0c8dbd1bj30og0y6tce.jpg)

 #### **primary key** **主键**

- ClickHouse 中的主键，和其他数据库不太一样，**它只提供了数据的一级索引，但是却不**

**是唯一约束。**这就意味着是可以存在相同 primary key 的数据的。

- 主键的设定主要依据是查询语句中的 where 条件。

- 根据条件通过对主键进行某种形式的二分查找，能够定位到对应的 index granularity（粒度）,避免了全表扫描。

  > index granularity： 直接翻译的话就是索引粒度，指在稀疏索引中两个相邻索引对应数据的间隔。ClickHouse 中的 MergeTree 默认是 8192。官方不建议修改这个值，除非该列存在大量重复值，比如在一个分区中几万行才有一个不同数据。

  **稀疏索引：**

  ![](https://tva1.sinaimg.cn/large/008i3skNly1gz0cf9ividj31460nigq3.jpg)

  稀疏索引的好处就是可以用很少的索引数据，定位更多的数据，代价就是只能定位到索	引粒度的第一行，然后再进行进行一点扫描。

  > 比如要查找id为15151的数据，可以判断出他大于10101 小于32343 ，因此会在这个区域内进行查找，类似的根据二分查找定位到具体的值。

#### **order by**

- order by 设定了分区内的数据按照哪些字段顺序进行有序保存。

- order by 是 MergeTree 中唯一一个必填项，甚至比 primary key 还重要，因为当用户不

  设置主键的情况，很多处理会依照 order by 的字段进行处理（比如后面会讲的去重和汇总）。

- **要求：主键必须是 order by 字段的前缀字段**。比如 order by 字段是 (id,sku_id) 那么主键必须是 id 或者(id,sku_id)

### 跳数索引

- MergeTree支持二级索引，又称之为跳数索引，是由数据的聚合信息构建而成。跳数索引的目录也是帮助查询，减少数据扫描的范围。

- 此索引在 `CREATE` 语句的列部分里定义。

  ```sql
  INDEX index_name expr TYPE type(...) GRANULARITY granularity_value
  ```

  - `*MergeTree` 系列的表可以指定跳数索引。

  - 跳数索引是指数据片段按照粒度(建表时指定的`index_granularity`)分割成小块后，将上述SQL的granularity_value数量的小块组合成一个大的块，对这些大块写入索引信息。

  - 这样有助于使用`where`筛选时跳过大量不必要的数据，减少`SELECT`需要读取的数据量。

  - 如：创建测试表

    ```sql
    create table t_order_mt2(
    id UInt32,	
    sku_id String,
    total_amount Decimal(16,2),
    create_time Datetime,
    INDEX a total_amount TYPE minmax GRANULARITY 5
    ) engine =MergeTree
    partition by toYYYYMMDD(create_time)
    primary key (id)
    order by (id, sku_id);
    ```

    > 其中 GRANULARITY N 是设定二级索引对于一级索引粒度的粒度。
    >
    > 即一级索引为[1,3],[3,6],[6,9] ，对应的二级索引范围为[1,9]，将3个一级索引进行了合并，这样查找时就不需要根据一级索引去进行比较，直接根据二级索引就可以。

>  目前在 ClickHouse 的官网上二级索引的功能在 v20.1.2.4 之前是被标注为实验性的，在这个版本之后默认是开启的。 

### **数据** **TTL**

- TTL 即 Time To Live，TTL用于设置值的生命周期，它既可以为整张表设置，也可以为每个列字段单独设置。表级别的 TTL 还会指定数据在磁盘和卷上自动转移的逻辑（也就是数据过期后可以移动到指定磁盘上）。

- TTL 表达式的计算结果必须是 [日期](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree/) 或 [日期时间](https://clickhouse.com/docs/zh/engines/table-engines/mergetree-family/mergetree/) 类型的字段。

- `TTL`子句不能被用于主键字段。

  示例：

  ```sql
  TTL time_column
  TTL time_column + interval
  ```

#### 1.**列级别** **TTL**

当列中的值过期时, ClickHouse会将它们替换成该列数据类型的默认值。如果数据片段中列的所有值均已过期，则ClickHouse 会从文件系统中的数据片段中删除此列。

##### 语法

创建表时指定 `TTL`

```sql
CREATE TABLE example_table
(
    d DateTime,
    a Int TTL d + INTERVAL 1 MONTH,
    b Int TTL d + INTERVAL 1 MONTH,
    c String
)
ENGINE = MergeTree
PARTITION BY toYYYYMM(d)
ORDER BY d;
```

为表中已存在的列字段添加 `TTL`

```sql
ALTER TABLE example_table
    MODIFY COLUMN
    c String TTL d + INTERVAL 1 DAY;
```

**案例演示**

- 创建测试表

  ```sql
  create table t_order_mt7(
  id UInt32,
  sku_id String,
  total_amount Decimal(16,2) TTL create_time+interval 10 SECOND,
  create_time Datetime 
  ) engine =MergeTree
  partition by toYYYYMMDD(create_time)
  primary key (id)
  order by (id, sku_id);
  ```

- 插入数据（注意：根据实际时间改变）

```sql
insert into t_order_mt7 values
(106,'sku_001',1000.00,'2022-02-03 10:44:32'),
(107,'sku_002',2000.00,'2022-02-03 10:44:32'),
(110,'sku_003',600.00,'2022-02-04 19:20:30');
```

![](https://tva1.sinaimg.cn/large/008i3skNly1gz0ik5e2dtj30ns0fe0u6.jpg)

- 手动合并，查看效果 到期后，指定的字段数据归 0

  ```sql
   optimize table t_order_mt7 final;
  ```

![](https://tva1.sinaimg.cn/large/008i3skNly1gz0ilgoy7mj30tc120wi1.jpg)

#### 2.表 TTL

- 表可以设置一个用于移除过期行的表达式，以及多个用于在磁盘或卷上自动转移数据片段的表达式。当表中的行过期时，ClickHouse 会删除所有对应的行。对于数据片段的转移特性，必须所有的行都满足转移条件。

- 下面的这条语句是数据会在 create_time 之后 10 秒丢失

  ```sql
  alter table t_order_mt3 MODIFY TTL create_time + INTERVAL 10 SECOND;
  ```

##### 案例演示

![](https://tva1.sinaimg.cn/large/008i3skNly1gz0iqo7gd7j30ze0m476i.jpg)

## ReplacingMergeTree

- ReplacingMergeTree 是 MergeTree 的一个变种，它存储特性完全继承 MergeTree，只是多了一个去重的功能。 

- 尽管 MergeTree 可以设置主键，但是 primary key 其实没有唯一约束的功能。如果你想处理掉重复的数据，可以借助这个 ReplacingMergeTree。 

### **去重时机**

数据的去重只会在同一批插入（新版本）或合并的过程中出现。合并会在未知的时间在后台进行，所以你无法预

先作出计划。有一些数据可能仍未被处理。

### **去重范围**

- 如果表经过了分区，去重只会在分区内部进行去重，不能执行跨分区的去重。

- 所以 ReplacingMergeTree 能力有限， ReplacingMergeTree 适用于在后台清除重复的数据以节省空间，但是它不保证没有重复的数据出现。

### **案例演示**

（1）创建表

```sql
create table t_order_rmt(
id UInt32,
sku_id String,
total_amount Decimal(16,2) ,
create_time Datetime 
) engine =ReplacingMergeTree(create_time)
partition by toYYYYMMDD(create_time)
primary key (id)
order by (id, sku_id);
```

- ReplacingMergeTree() 填入的参数为版本字段，重复数据保留版本字段值最大的。

- 如果不填版本字段，默认按照插入顺序保留最后一条。

（2）向表中插入数据

```sql
insert into t_order_rmt values
(101,'sku_001',1000.00,'2020-06-01 12:00:00') ,
(102,'sku_002',2000.00,'2020-06-01 11:00:00'),
(102,'sku_004',2500.00,'2020-06-01 12:00:00'),
(102,'sku_002',2000.00,'2020-06-01 13:00:00'),
(102,'sku_002',12000.00,'2020-06-01 13:00:00'),
(102,'sku_002',600.00,'2020-06-02 12:00:00');
```

（3）执行第一次查询

可以看到插入的数据是根据id和sku_id 去重过的。

```sql
select * from t_order_rmt;
```

![](https://tva1.sinaimg.cn/large/008i3skNly1gz0of9u8h5j30ps0j00ui.jpg)

（4）再次插入数据

```sql
insert into t_order_rmt values
(101,'sku_001',1000.00,'2020-06-01 12:00:00') ,
(102,'sku_002',2000.00,'2020-06-01 11:00:00'),
(102,'sku_004',2500.00,'2020-06-01 12:00:00'),
(102,'sku_002',2000.00,'2020-06-01 13:00:00'),
(102,'sku_002',12000.00,'2020-06-01 13:00:00'),
(102,'sku_002',600.00,'2020-06-02 12:00:00');
```

(5) 再次查询，可以看到这次依旧只插入了4条数据，但是和第一次插入的数据没有进行去重，依旧存在重复数据。

![](https://tva1.sinaimg.cn/large/008i3skNly1gz0olyjbfej30oo0qgjuc.jpg)

（6）手动合并

```sql
OPTIMIZE TABLE t_order_rmt FINAL;
```

  (7) 再执行一次查询，此时数据已经根据分区进行了最终去重。

![](https://tva1.sinaimg.cn/large/008i3skNly1gz0ovmauddj30ow0imjt6.jpg)

### 案例结论

➢ 实际上是使用 order by 字段作为唯一键

➢ 去重不能跨分区

➢ 只有同一批插入（新版本）或合并分区时才会进行去重

➢ 认定重复的数据保留，版本字段值最大的

➢ 如果版本字段相同则按插入顺序保留最后一笔

## **SummingMergeTree**

- 对于不查询明细，只关心以维度进行汇总聚合结果的场景。如果只使用普通的MergeTree的话，无论是存储空间的开销，还是查询时临时聚合的开销都比较大。

- ClickHouse 为了这种场景，提供了一种能够“预聚合”的引擎 SummingMergeTree
- 不支持幂等性，当写入100条，写了50条后挂了，后面再次同步100条数据时，会把前50条的数据聚合两次。

### **案例演示**

（1）创建表

```sql
create table t_order_smt(
id UInt32,
sku_id String,
total_amount Decimal(16,2) ,
create_time Datetime 
) engine =SummingMergeTree(total_amount)
partition by toYYYYMMDD(create_time)
primary key (id)
order by (id,sku_id );
```

（2）插入数据

```sql
insert into t_order_smt values
(101,'sku_001',1000.00,'2020-06-01 12:00:00'),
(102,'sku_002',2000.00,'2020-06-01 11:00:00'),
(102,'sku_004',2500.00,'2020-06-01 12:00:00'),
(102,'sku_002',2000.00,'2020-06-01 13:00:00'),
(102,'sku_002',12000.00,'2020-06-01 13:00:00'),
(102,'sku_002',600.00,'2020-06-02 12:00:00');
```

（3）执行第一次查询，可以看到已经做了一次聚合

```sql
select * from t_order_smt;
```

![](https://tva1.sinaimg.cn/large/008i3skNly1gz0p639ls5j30py0igabu.jpg)

（4）再次插入一条重复数据，并查询。

```sql
insert into t_order_smt values
(101,'sku_001',2000.00,'2020-06-01 13:00:00')
```

![](https://tva1.sinaimg.cn/large/008i3skNly1gz0p8yhgnfj30pq0liq5h.jpg)

（5）手动合并,并再次执行查询，可以看到数据已经全部聚合

```sql
OPTIMIZE TABLE t_order_smt FINAL;
```

![](https://tva1.sinaimg.cn/large/008i3skNly1gz0pau5gw1j30oq0qm0ve.jpg)

### 案例结论

➢ 以 SummingMergeTree（）中指定的列作为汇总数据列 

➢ 以 order by 的列为准，作为维度列

➢ 可以填写多列必须数字列，如果不填，以所有非维度列且为数字列的字段为汇总数据列

➢ 其他的列按插入顺序保留第一行

➢ 不在一个分区的数据不会被聚合

➢ 只有在同一批次插入(新版本)或分片合并时才会进行聚合

### 能不能直接执行查询 SQL 得到汇总值？

不行，可能会包含一些还没来得及聚合的临时明细如果要是获取汇总值，还是需要使用 sum 进行聚合，这样效率会有一定的提高，但本身 ClickHouse 是列式存储的，效率提升有限，不会特别明显。

```sql
select total_amount from XXX where province_name=’’ and create_date=’xxx’
```

改写为：

```sql
select sum(total_amount) from province_name=’’ and create_date=‘xxx’
```



# 集成

## MySQL

MySQL 引擎可以对存储在远程 MySQL 服务器上的数据执行 `SELECT` 查询和`insert`插入。

```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1] [TTL expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2] [TTL expr2],
    ...
) ENGINE = MySQL('host:port', 'database', 'table', 'user', 'password'[, replace_query, 'on_duplicate_clause'])
SETTINGS
    [connection_pool_size=16, ]
    [connection_max_tries=3, ]
    [connection_wait_timeout=5, ] /* 0 -- do not wait */
    [connection_auto_close=true ]
;
```

更多见：[MySQL | ClickHouse Documentation](https://clickhouse.com/docs/en/engines/table-engines/integrations/mysql/)

# 日志系列

- 这些引擎是为了需要写入许多小数据量（少于一百万行）的表的场景而开发的。
- 这系列的引擎有：
  - [StripeLog](https://clickhouse.com/docs/zh/engines/table-engines/log-family/stripelog/)
  - [日志](https://clickhouse.com/docs/zh/engines/table-engines/log-family/log/)
  - [TinyLog](https://clickhouse.com/docs/zh/engines/table-engines/log-family/tinylog/)

## TinyLog

以列文件的形式保存在磁盘上，不支持索引，没有并发控制。一般保存少量数据的小表，

生产环境上作用有限。可以用于平时练习测试用。

##  

