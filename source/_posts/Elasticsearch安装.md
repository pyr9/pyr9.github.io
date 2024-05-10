---
title: Elasticsearch安装
date: 2022-02-05 20:08:30
tags: 分布式框架
categories: ElasticSearch
---

# Elasticsearch安装

1.下载es.  Es 下载链接： [Download Elasticsearch | Elastic](https://www.elastic.co/cn/downloads/elasticsearch)

![image-20230228230445183](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228230445183.png)

2.解压后在bin路径下，运行

```sql
➜  bin ./elasticsearch -d
```

3.访问[localhost:9200](http://localhost:9200/),看到下面的页面即启动成功

![image-20230228230454065](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228230454065.png)

# Kibana安装

1. Kibana 下载链接：[Download Kibana Free | Get Started Now | Elastic](https://www.elastic.co/cn/downloads/kibana)

![image-20230228230502701](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228230502701.png)

2. 修改 kibana.yml 

```sql
server.port: 5601
server.host: "localhost"
elasticsearch.hosts: ["http://localhost:9200"]
```

3.访问http://localhost:5601/app/kibana, 看到下面的页面即启动成功

![image-20230228230512243](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228230512243.png)

# 安装IK分词器

我们后续也需要使用Elasticsearch来进行中文分词，所以需要单独给Elasticsearch安装IK分词器插件。以下为具体安装步骤：

1. 下载ElasticsearchIK分词器 https://github.com/medcl/elasticsearch-analysis-ik/releases

2.  在es的安装目录下/plugins创建ik，将下载的ik分词器上传并解压到该目录

3.  重启Elasticsearch

   ```sql
   查找进程命令 ps -ef | grep elastic
   杀掉进程 kill -9 2382（进程号）
   ```

4.  测试分词效果，访问[Dev Tools - Elastic](http://localhost:5601/app/dev_tools#/console)

   ```sql
   POST _analyze
   {
   "analyzer":"standard",
   "text":"我爱你中国"
   }
   ```

   ![image-20230228230530722](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228230530722.png)

   

   - ik_smart:会做最粗粒度的拆分

   ```sql
   POST _analyze
   {
   "analyzer":"ik_smart",
   "text":"我爱你中国"
   }
   ```

   ![image-20230228230546771](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228230546771.png)

   - k_max_word:会将文本做最细粒度的拆分

   ```sql
   POST _analyze
   {
   "analyzer":"ik_max_word",
   "text":"我爱你中国"
   }
   ```

   ![image-20230228230556755](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228230556755.png)

   

# 指定IK分词器作为默认分词器

​        ES的默认分词设置是standard，这个在中文分词时就比较尴尬了，会单字拆分，比如我搜索关键词“清华大学”，这时候会按“清”，“华”，“大”，“学”去分词，然后搜出来的都是些“清清的河水”，“中华儿女”，“地大物博”，“学而不思则罔”之类的莫名其妙的结果，

​        这里我们就想把这个分词方式修改一下，于是呢，就想到了ik分词器，有两种ik_smart和ik_max_word。ik_smart会将“清华大学”整个分为一个词，而ik_max_word会将“清华大学”分为“清华大学”，“清华”和“大学”，按需选其中之一就可以了。修改默认分词方法(这里修改school_index索引的默认分词为：ik_max_word)：

```sql
PUT /school_index
{
  "settings":{
    "index":{
      "analysis.analyzer.default.type":"ik_max_word"
    }
  }
}
```

![image-20230228230604782](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228230604782.png)
