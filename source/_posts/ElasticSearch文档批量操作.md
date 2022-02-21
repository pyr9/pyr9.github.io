---
title: ElasticSearch文档批量操作
date: 2022-02-21 21:53:11
tags: 分布式框架
categories: ElasticSearch
---

# **文档批量操作**

## 批量获取文档数据 

批量获取文档数据是通过_mget的API来实现的 

### 在URL中不指定index和type

```sql
GET _mget
{
  "docs": [
    {
      "_index": "es_db",
      "_type": "_doc",
      "_id": 1
    },
    {
      "_index": "es_db",
      "_type": "_doc",
      "_id": 2
    }
  ]
}
```

### 在URL中指定index

```sql
GET /es_db/_mget
{
  "docs": [
    {
      "_type": "_doc",
      "_id": 1
    },
    {
      "_type": "_doc",
      "_id": 2
    }
  ]
}
```

### 在URL中指定index和type

```sql
GET /es_db/_doc/_mget
{
  "docs": [
    {
      "_id": 1
    },
    {
      "_id": 2
    }
  ]
}
```

## 批量操作文档数据 

### 批量创建文档create

```sql
POST _bulk
{
  "create": {
    "_index": "article",
    "_type": "_doc",
    "_id": 3
  }
}
{
  "id": 3,
  "title": "文章1",
  "content": "内容1",
  "tags": [
    "tag1",
    "tag2"
  ],
  "create_time": 1554015482530
}
{
  "create": {
    "_index": "article",
    "_type": "_doc",
    "_id": 4
  }
}
{
  "id": 4,
  "title": "文章2",
  "content": "内容2",
  "tags": [
    "tag3",
    "tag4"
  ],
  "create_time": 1554015482530
}
```

### 普通创建或全量替换index

```sql
POST _bulk
{
  "index": {
    "_index": "article",
    "_type": "_doc",
    "_id": 3
  }
}
{
  "id": 3,
  "title": "更新后的文章1",
  "content": "更新后的内容1",
  "tags": [
    "tag1",
    "tag2"
  ],
  "create_time": 1554015482530
}
{
  "index": {
    "_index": "article",
    "_type": "_doc",
    "_id": 5
  }
}
{
  "id": 5,
  "title": "文章5",
  "content": "内容5",
  "tags": [
    "tag5",
    "tag14"
  ],
  "create_time": 1554015482530
}
```

### 批量修改update

```sql
POST _bulk
{
  "update": {
    "_index": "article",
    "_type": "_doc",
    "_id": 3
  }
}
{
  "doc": {
    "title": "更新后的文章11",
    "content": "更新后的内容11"
  }
}
{
  "update": {
    "_index": "article",
    "_type": "_doc",
    "_id": 5
  }
}
{
  "doc": {
    "title": "文章51",
    "content": "内容51"
  }
}
```

### 批量删除delete

```sql
POST _bulk
{
  "delete": {
    "_index": "article",
    "_type": "_doc",
    "_id": 3
  }
}
{
  "delete": {
    "_index": "article",
    "_type": "_doc",
    "_id": 4
  }
}s
```

