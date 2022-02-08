---
title: Elasticsearch安装
date: 2022-02-05 20:08:19
tags: 分布式框架
categories:Elasticsearch
---

# Elasticsearch安装

1.下载es.  Es 下载链接： [Download Elasticsearch | Elastic](https://www.elastic.co/cn/downloads/elasticsearch)

<img src="/Users/panyurou/Library/Application Support/typora-user-images/image-20220205235202033.png" alt="image-20220205235202033" style="zoom:50%;" />

2.解压后在bin路径下，运行

```sql
➜  bin ./elasticsearch -d
```

3.访问[localhost:9200](http://localhost:9200/),看到下面的页面即启动成功

![image-20220205235359603](/Users/panyurou/Library/Application Support/typora-user-images/image-20220205235359603.png)

# Kibana安装

1. Kibana 下载链接：[Download Kibana Free | Get Started Now | Elastic](https://www.elastic.co/cn/downloads/kibana)

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gz32m89db4j30u00yigof.jpg" style="zoom:50%;" />

2. 修改 kibana.yml 

```sql
server.port: 5601
server.host: "localhost"
elasticsearch.hosts: ["http://localhost:9200"]
```

3.访问http://localhost:5601/app/kibana, 看到下面的页面即启动成功

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gz32vfqn6kj31be0u00vp.jpg" style="zoom:50%;" />
