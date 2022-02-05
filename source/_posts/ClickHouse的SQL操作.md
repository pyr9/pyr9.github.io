---
title: ClickHouse的SQL操作
date: 2022-02-03 22:50:09
tags: 数据库
categories: clickhouse
---

# ClickHouse的SQL操作

基本上来说传统关系型数据库（以 MySQL 为例）的 SQL 语句，ClickHouse 基本都支持，这里不会从头讲解 SQL 语法只介绍 ClickHouse 与标准 SQL（MySQL）不一致的地方

## Insert

基本与标准 SQL（MySQL）基本一致

（1）标准

```sql
insert into [table_name] values(…),(….)
```

（2）从表到表的插入

```sql
insert into [table_name] select a,b,c from [table_name_2]
```

## Update 和Delete

- ClickHouse 提供了 Delete 和 Update 的能力，这类操作被称为 Mutation 查询，它可以看做 Alter 的一种。

- 虽然可以实现修改和删除，但是和一般的 OLTP 数据库不一样，**Mutation** 语句是一种很重的操作，而且不支持事务。

  > “重”的原因主要是每次修改或者删除都会导致放弃目标数据的原有分区，重建新分区。合并的时候才会删除原有分区，所以尽量做批量的变更，不要进行频繁小数据的操作。

### 删除操作

```sql
alter table t_order_smt delete where sku_id ='sku_001';
```

### 修改操作

```sql
alter table t_order_smt update total_amount=toDecimal32(2000.00,2) where id =102;
```

### 开发建议

clickhouse虽然支持 Delete 和 Update 的能力，但是这类操作很重，一般建议不要使用，如果要使用，都可以采用增加数据的方式来实现，如更新的话，可以给列新加版本号；删除的话，可以增加一列删除标记为：delete_at，但这样会造成数据膨胀，可以定期删除失效的数据。

## 查询

ClickHouse 基本上与标准 SQL 差别不大

➢ 支持子查询

➢ 支持 CTE(Common Table Expression 公用表表达式 with 子句) 

➢ 支持各种 JOIN，但是 JOIN 操作无法使用缓存，所以即使是两次相同的 JOIN 语句，ClickHouse 也会视为两条新 SQL

➢ 窗口函数

➢ 不支持自定义函数

➢ GROUP BY 操作增加了 with rollup\with cube\with total 用来计算小计和总计。

**演示**

（1）插入数据

```sql
bfdf602492c9 :)  alter table t_order_mt delete where 1=1;
bfdf602492c9 :) insert into t_order_mt values
                (101,'sku_001',1000.00,'2020-06-01 12:00:00'),
                (101,'sku_002',2000.00,'2020-06-01 12:00:00'),
                (103,'sku_004',2500.00,'2020-06-01 12:00:00'),
                (104,'sku_002',2000.00,'2020-06-01 12:00:00'),
                (105,'sku_003',600.00,'2020-06-02 12:00:00'),
                (106,'sku_001',1000.00,'2020-06-04 12:00:00'),
                (107,'sku_002',2000.00,'2020-06-04 12:00:00'),
                (108,'sku_004',2500.00,'2020-06-04 12:00:00'),
                (109,'sku_002',2000.00,'2020-06-04 12:00:00'),
                (110,'sku_003',600.00,'2020-06-01 12:00:00');
```

（2）with rollup：从右至左去掉维度进行小计

可以看出分别根据  【id,sku_id】【id】【(空)】 进行了分组

```sql
bfdf602492c9 :) select id , sku_id,sum(total_amount) from t_order_mt group by id,sku_id with rollup;
```

![](https://tva1.sinaimg.cn/large/008i3skNly1gz1rqitalcj30hg0tamz1.jpg)

（3）with cube : 从右至左去掉维度进行小计，再从左至右去掉维度进行小计

可以看出分别根据  【id,sku_id】【id】【sku_id】【(空)】 进行了分组

![](https://tva1.sinaimg.cn/large/008i3skNly1gz1rt0556dj30hu0x0di6.jpg)

（4）with totals: 只计算合计

```sql
bfdf602492c9 :)  select id , sku_id,sum(total_amount) from t_order_mt group by
                id,sku_id with totals;
```

![](https://tva1.sinaimg.cn/large/008i3skNly1gz1rutq1qaj30j20im75i.jpg)

## alter操作

同 MySQL 的修改字段基本一致

1）新增字段

```sql
alter table tableName add column newcolname String after col1; 
```

2）修改字段类型

```sql
alter table tableName modify column newcolname String; 
```

3）删除字段

```sql
alter table tableName drop column newcolname; 
```

> 更多语法参考：[ClickHouse Documentation](https://clickhouse.com/docs/en/sql-reference/)

