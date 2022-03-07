---
title: Elasticsearch数据管理
date: 2022-02-12 23:02:34
tags: 分布式框架
categories: ElasticSearch
---

# ES数据管理 

## ES数据管理概述 

- ES是面向文档(document oriented)的，这意味着它可以存储整个对象或文档 (document)。
- 然而它不仅仅是存储，还会索引(index)每个文档的内容使之可以被搜索。
- 在ES中，你可以对文档（而非成行成列的数据）进行索引、搜索、排序、过滤。
- ES使用JSON作为文档序列化格式。 

**ES存储的一个员工文档的格式示例：** 

```sql
 { 
  "email": "584614151@qq.com",
  "name": "张三", 
  "age": 30,
  "interests": [ "篮球", "健身" ]
 }
```

> JSON现在已经被大多语言所支持，而且已经成为NoSQL领域的标准格式。 

## 基本操作 

### 创建索引 

格式: PUT /索引名称

```sql
PUT /es_db
```

### 查询索引 

格式: GET /索引名称 

```sql
GET /es_db
```

###  删除索引 

格式: DELETE /索引名称

```sql
DELETE /es_db
```

###  添加文档 

- 格式: POST /索引名称/类型/id(可选，不传会创建默认的id)

```sql
POST /es_db/_doc
{
  "name": "张三",
  "sex": 1,
  "age": 25,
  "address": "广州天河公园",
  "remark": "java developer"
}
```

- 格式: PUT /索引名称/类型/id 

```sql
PUT /es_db/_doc/1
{
  "name": "张三",
  "sex": 1,
  "age": 25,
  "address": "广州天河公园",
  "remark": "java developer"
}
```

### 修改文档 

- 格式: PUT /索引名称/类型/id 

```sql
PUT /es_db/_doc/1
{
  "name": "张三",
  "sex": 1,
  "age": 25,
  "address": "广州天河公园",
  "remark": "java developer"
}
```

### 查询文档 

格式: GET /索引名称/类型/id 

```sql
 GET /es_db/_doc/1
```

### 删除文档 

格式: DELETE /索引名称/类型/id 

```sql
DELETE /es_db/_doc/1
```

###  查询操作 

#### 查询当前类型中的所有文档 _search 

 格式: GET /索引名称/类型/_search 

举例：

```sql
GET /es_db/_doc/_search 
```

#### 	条件查询

- 格式: GET /索引名称/类型/_search?q=*:*** 

- SQL: select * from student where age = 28 

举例：如要查询age等于28岁的 _search?q=*:*** 

```sql
GET /es_db/_doc/_search?q=age:28 
```

#### 范围查询

##### in

- 格式: GET /索引名称/类型/_search?q=***[25 TO 26] 
-  SQL: select * from student where age between 25 and 26 

举例：如要查询age在25至26岁之间的 

```sql
GET /es_db/_doc/_search?q=age[25 TO 26] 
```

**注意: TO 必须为大写** 

##### 大于

-  格式:  GET /索引名称/类型/_search?q=age:>** 
- SQL: select * from student where age > 28

举例：查询年龄大于28的 

```sql
 GET /es_db/_doc/_search?q=age:>28
```

##### 小于等于

- 格式: GET /索引名称/类型/_search?q=age:<=** 
- SQL: select * from student where age <= 28 

举例：查询年龄小于等于28岁的

```sql
GET /es_db/_doc/_search?q=age:<=28
```

#### 批量查询

- 格式: GET /索引名称/类型/_mget 
-  SQL: select * from student where id in (1,2) 

举例：根据多个ID进行批量查询

```sql
 GET /es_db/_doc/_mget
 {
   "ids":["3","2"]
 }
```

#### 分页查询

- 格式: GET /索引名称/类型/_search?q=age[25 TO 26]&from=0&size=1 

- SQL: select * from student where age between 25 and 26 limit 0, 1

举例：

```sql
 GET /es_db/_doc/_search?q=age[25 TO 26]&from=0&size=1
```

#### 查询结果只输出某些字段

- 格式: GET /索引名称/类型/_search?_source=字段,字段
- SQL: select name,age from student 

举例：

```sql
GET /es_db/_doc/_search?_source=name,age
```

#### 查询结果排序

- 格式: GET /索引名称/类型/_search?sort=字段 desc 
- SQL: select * from student order by age desc 

举例：

```sql
GET /es_db/_doc/_search?sort=age:desc
```
