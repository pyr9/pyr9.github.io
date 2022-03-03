---
title: ElasticSearch-DSL语言高级查询
date: 2022-02-22 22:23:32
tags: 分布式框架
categories: ElasticSearch
---

# DSL语言高级查询

- Elasticsearch提供了基于JSON的DSL来定义查询。 
- DSL由叶子查询子句和复合查询子句两种子句组成

> DSL(Domain Specific Language)领域专用语言 

## 无查询条件 

无查询条件是查询所有，默认是查询所有的，或者使用match_all表示所有 

```java
GET /es_db/_doc/_search
{
  "query": {
    "match_all": {}
  }
}
```

## 有查询条件

### 模糊匹配 

- 模糊匹配主要是针对文本类型的字段

- 文本类型的字段会对内容进行分词，对查询时，也会对搜索条件进行分词，然后通过倒排索引查找到匹配的数据
- 模糊匹配主要通过match等参数来实现 
  - match : 通过match关键词模糊匹配条件内容,match会根据该字段的分词器，进行分词查询
  - prefix : 前缀匹配
  - regexp : 通过正则表达式来匹配数据 

 **举例:**

```java
POST /es_db/_doc/_search
{
  "from": 0,
  "size": 2,
  "query": {
    "match": {
      "address": "广州"
    }
  }
}
```

> SQL: select * from user where address like '%广州%' limit 0, 2 

- 多字段模糊匹配查询 multi_match 

```java
POST /es_db/_doc/_search
{
  "from": 0,
  "size": 2,
  "query": {
    "multi_match": {
      "query": "广州公园",
      "fields": [
        "address",
        "name"
      ]
    }
  }
}
```

> SQL: select * from student where name like '%广州公园%' or address like '%广州公园%' 

- 未指定字段条件查询 query_string , 含 AND 与 OR 条件

```sql
POST /es_db/_doc/_search
{
  "query": {
    "query_string": {
      "query": "(广州) OR 长沙"
    }
  }
}
```

- 指定字段条件查询 query_string , 含 AND 与 OR 条件 

```java
POST /es_db/_doc/_search
{
  "query": {
    "query_string": {
      "query": "(广州) OR 长沙",
      "fields":["name","address"]
    }
  }
}
```

- 范围查询
  - range：范围关键字 
  - gte 大于等于
  - lte 小于等于 
  - gt 大于 
  - lt 小于
  - now 当前时间

```java
POST /es_db/_doc/_search
{
  "query":{
    "range" : {
      "age" : {
        "gte":25,
        "lte":28
      }
    }
  }
}
```

> SQL: select * from user where age between 25 and 28 

- 分页、输出字段、排序综合查询

```java
POST /es_db/_doc/_search
{
    "query":{
    "range" : {
      "age" : {
        "gte":25,
        "lte":28
      }
    }
  },
  "from": 0,
  "size": 2,
  "_source": ["name", "age", "book"],
  "sort": {"age":"desc"}
}
```

## Filter过滤器方式查询

它的查询不会计算相关性分值，也不会对结果进行排序, 因此效率会高一点，查询的结果可以被缓存

```java
POST /es_db/_doc/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "age": 25
        }
      }
    }
  }
}
```

## 总结

- match: 模糊匹配，需要指定字段名，但是输入会进行分词，比如"hello world"会进行拆分为hello和world，然后匹配，如果字段中包含hello或者 world，或者都包含的结果都会被查询出来，也就是说match是一个部分匹配的模糊查询。查询条件相对来说比较宽松。 

- term：这种查询和match在有些时候是等价的，比如我们查询单个的词hello，那么会和match查询结果一样，但是如果查询"hello world"，结果就相差很大，因为这个输入不会进行分词，就是说查询的时候，是查询字段分词结果中是否有"hello world"的字样，而不是查询字段中包含"hello world"的字样。当保存 

  数据"hello world"时，elasticsearch会对字段内容进行分词，"hello world"会被分成hello和world，不存在"hello world"，因此这里的查询结果会为空。这也是term查询和match的区别。

- query_string：和match类似，但是match需要指定字段名，query_string是在所 

  有字段中搜索，范围更广泛。
