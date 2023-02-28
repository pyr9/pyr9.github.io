---
title: ElasticSearch文档映射
date: 2022-03-03 22:07:42
tags: 分布式框架
categories: ElasticSearch
---

# 文档映射

## 映射类型

ES中映射可以分为动态映射和静态映射 

### 动态映射： 

在关系数据库中，需要事先创建数据库，然后在该数据库下创建数据表，并创建 表字段、类型、长度、主键等，最后才能基于表插入数据。而Elasticsearch中不需要定义Mapping映射（即关系型数据库的表、字段等），在文档写入Elasticsearch时，会根据文档字段自动识别类型，这种机制称之为动态映射。 

动态映射规则如下： 

![image-20230228233355466](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228233355466.png)

**操作示例**

- 创建文档(ES根据数据类型, 会自动创建映射) 

```java
POST /es_db1/_doc/32
{
  "name": "王武",
  "sex": 1,
  "age": 28,
  "address": "",
  "remark": "java developer"
}
```

-  获取文档映射 

```java
GET /es_db1/_mapping
```

结果如下：

```java
{
  "es_db1" : {
    "mappings" : {
      "properties" : {
        "address" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "age" : {
          "type" : "long"
        },
        "name" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "remark" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "sex" : {
          "type" : "long"
        }
      }
    }
  }
}
```

### 静态映射： 

静态映射是在Elasticsearch中也可以事先定义好映射，包含文档的各字段类型、分词器等，这种方式称之为静态映射。 

**操作示例**

- 创建索引和设置文档映射 
  - index: 是否分词存储
  - store: 是否存储在库里

```java
PUT /es_db4
{
  "mappings": {
    "properties": {
      "name": {
        "type": "keyword",
        "index": true,
        "store": true
      },
      "sex": {
        "type": "integer",
        "index": true,
        "store": true
      },
      "age": {
        "type": "integer",
        "index": true,
        "store": true
      },
      "book": {
        "type": "text",
        "index": true,
        "store": true
      },
      "address": {
        "type": "text",
        "index": true,
        "store": true
      }
    }
  }
}
```

结果：

```java
{
  "es_db4" : {
    "mappings" : {
      "properties" : {
        "address" : {
          "type" : "text",
          "store" : true
        },
        "age" : {
          "type" : "integer",
          "store" : true
        },
        "book" : {
          "type" : "text",
          "store" : true
        },
        "name" : {
          "type" : "keyword",
          "store" : true
        },
        "sex" : {
          "type" : "integer",
          "store" : true
        }
      }
    }
  }
}
```

- 根据静态映射创建文档

```java
PUT /es_db/_doc/4
{
  "name": "Jack",
  "sex": 1,
  "age": 25,
  "book": "elasticSearch入门至精通",
  "address": "广州车陂"
}
```

- 获取文档映射 

````=java
 GET /es_db4/_mapping
````

## 文档核心类型（Core datatype）

### 类型分类

- 字符串：string，string类型包含 text 和 keyword。 
- text：该类型被用来索引长文本，在创建索引前会将这些文本进行分词，转化为词的组合，建立索引；允许es来检索这些词，text类型不能用来排序和聚合。
- keyword：该类型不能分词，可以被用来检索过滤、排序和聚合，keyword类型不可用text进行分词模糊检索。
- 数值型：long、integer、short、byte、double、float 
- 日期型：date 
- 布尔型：boolean 

**通过term 和 match查询数据时细节点以及数据类型keyword与text区别**

- **term查询**
  - term查询keyword字段: term不会分词。而keyword字段也不分词。需要完全匹配才可
  - term查询text字段: 因为text字段会分词，而term不分词，所以term查询的条件必须是text字段分词后的某一个
- **match查询**
  - match会被分词，而keyword不会被分词，match的需要跟keyword的完全匹配可以。
  - match分词，text也分词，只要match的分词结果和text的分词结果有相同的就匹配

## 对已存在的mapping映射进行修改

- 重新建立一个静态索引 ,把之前索引里的数据导入到新的索引里 

```java
POST _reindex
{
  "source": {
    "index": "es_db"
  },
  "dest": {
    "index": "es_db_2"
  }
}
```

> 注：_reindex 会同时把之前索引里的数据导入到新的索引里 

- 删除原创建的索引

```hava
 DELETE /es_db
```

- 为新索引起个别名, 为原索引名 

```java
PUT /es_db_2/_alias/es_db
```

