---
title: docker搭建ELK
date: 2024-01-04 09:56:22
tags:
categories:
---

# 1. ELK是什么？

ELK主要由ElasticSearch、Logstash和Kibana三个开源工具组成，还有其他专门由于收集数据的轻量型数据采集器Beats。

- Elasticsearch ：分布式搜索引擎。具有高可伸缩、高可靠、易管理等特点。可以用于全文检索、结构化检索和分析，并能将这三者结合起来。Elasticsearch 是用Java 基于 Lucene 开发，现在使用最广的开源搜索引擎之一，Wikipedia 、StackOverflow、Github 等都基于它来构建自己的搜索引擎。在elasticsearch中，所有节点的数据是均等的。
- Logstash ：数据收集处理引擎。支持动态的从各种数据源搜集数据，并对数据进行过滤、分析、丰富、统一格式等操作，然后存储以供后续使用。
- Kibana ：可视化化平台。它能够搜索、展示存储在 Elasticsearch 中索引数据。使用它可以很方便的用图表、表格、地图展示和分析数据。
- Filebeat：轻量级数据收集引擎。相对于Logstash所占用的系统资源来说，Filebeat 所占用的系统资源几乎是微乎及微。它是基于原先 Logstash-fowarder 的源码改造出来。换句话说：Filebeat就是新版的 Logstash-fowarder，也会是 ELK Stack 在 Agent 的第一选择。

# 2. 安装

## 2.1 **创建docker网络**

```shell
docker network create -d bridge elastic
```

## 2.2 安装ES

### 1. 拉取镜像

```
docker pull elasticsearch:8.4.3
```

### 2. 创建挂载目录并授权

```
mkdir /etc/elk8.4.3/elasticsearch
sudo chown -R 1000:1000 /etc/elk8.4.3/elasticsearch
```

### 3. 第一次执行docker启动脚本

```
docker run -it \
    -p 9200:9200 \
    -p 9300:9300 \
    --name elasticsearch \
    --net elastic \
    -e ES_JAVA_OPTS="-Xms1g -Xmx1g" \
    -e "discovery.type=single-node" \
    -e LANG=C.UTF-8 \
    -e LC_ALL=C.UTF-8 \
    elasticsearch:8.4.3
```

### 4. 将容器内的文件复制到主机上

```
docker cp elasticsearch:/usr/share/elasticsearch/config /etc/elk8.4.3/elasticsearch
docker cp elasticsearch:/usr/share/elasticsearch/data /etc/elk8.4.3/elasticsearch
docker cp elasticsearch:/usr/share/elasticsearch/plugins /etc/elk8.4.3/elasticsearch
docker cp elasticsearch:/usr/share/elasticsearch/logs /etc/elk8.4.3/elasticsearch
```

### 5. 修改配置文件

`/etc/elk8.4.3/elasticsearch/config/elasticsearch.yml`

```
cluster.name: "docker-cluster"
network.host: 0.0.0.0

# Enable security features
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-headers: Authorization

xpack.security.enabled: true
xpack.security.enrollment.enabled: true
xpack.monitoring.collection.enabled: true


# Enable encryption for HTTP API client connections, such as Kibana, Logstash, and Agents
xpack.security.http.ssl:
  enabled: false
  keystore.path: certs/http.p12

# Enable encryption and mutual authentication between cluster nodes
xpack.security.transport.ssl:
  enabled: false
  verification_mode: certificate
  keystore.path: certs/transport.p12
  truststore.path: certs/transport.p12
```

### 6. 删除容器

```
docker rm /elasticsearch
```

### 7. 修改docker启动脚本，增加-v挂载目录

```
docker run -it \
    -d \
    -p 9200:9200 \
    -p 9300:9300 \
    --name elasticsearch \
    --net elastic \
    -e ES_JAVA_OPTS="-Xms1g -Xmx1g" \
    -e "discovery.type=single-node" \
    -e LANG=C.UTF-8 \
    -e LC_ALL=C.UTF-8 \
    -v /etc/elk8.4.3/elasticsearch/config:/usr/share/elasticsearch/config \
    -v /etc/elk8.4.3/elasticsearch/data:/usr/share/elasticsearch/data \
    -v /etc/elk8.4.3/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
    -v /etc/elk8.4.3/elasticsearch/logs:/usr/share/elasticsearch/logs \
    elasticsearch:8.4.3
```

### 8. 设置用户密码

- 进入容器 `docker exec -it elk /bin/bash`
- 进入bin目录 `cd bin`
- 给elastic，kibana_system 设置密码为“sith-mes” ,执行`./elasticsearch-setup-passwords interactive `或`elasticsearch-reset-password -u elastic -i`
- 重启容器

### 9. 配置Monitoring监控日志

避免实例的大部分空间被监控索引占用，可通过以下两种方式进行优化（实际使用中，可以将以上两种方案结合使用）

[如何配置Monitoring监控日志_检索分析服务 Elasticsearch版(ES)-阿里云帮助中心 (aliyun.com)](https://help.aliyun.com/zh/es/user-guide/configure-monitoring-indexes)

## 2.3 安装Kibana

### 1. 拉取镜像

```
docker pull kibana:8.4.3
```

### 2. 创建挂载目录并授权

```
mkdir /etc/elk8.4.3/kibana
sudo chown -R 1000:1000 /etc/elk8.4.3/kibana
```

### 3. 第一次执行docker启动脚本

```
docker run -it \
	-d \
	--restart=always \
	--log-driver json-file \
	--log-opt max-size=100m \
	--log-opt max-file=2 \
	--name kibana \
	-p 5601:5601 \
	--net elastic \
	kibana:8.4.3
```

### 4. 将容器内的文件复制到主机上

```
docker cp kibana:/usr/share/kibana/config /etc/elk8.4.3/kibana/        
docker cp kibana:/usr/share/kibana/data /etc/elk8.4.3/kibana/        
docker cp kibana:/usr/share/kibana/plugins /etc/elk8.4.3/kibana/ 
docker cp kibana:/usr/share/kibana/logs /etc/elk8.4.3/kibana/  
```

### 5. 修改配置文件

 `/etc/elk8.4.3/kibana/config/kibana.yml`

```
server.host: "0.0.0.0"
server.shutdownTimeout: "5s"
elasticsearch.hosts: [ "http://192.168.3.17:9200" ]
elasticsearch.username: "kibana_system"
elasticsearch.password: "sith-mes"
monitoring.ui.container.elasticsearch.enabled: true
i18n.locale: "en"
xpack.encryptedSavedObjects.encryptionKey: dcbf819d8874e8242eaf107d538fe874
xpack.reporting.encryptionKey: ba2d98e6dad4fa73d77f5f34a568cfdd
xpack.security.encryptionKey: 3871dfc1bd945e1141364e4244800b39
```

⚠️**这里的用户是 kibana_system**

### 6. 删除容器

```
docker rm /kibana
```

### 7. 修改docker启动脚本，增加-v挂载目录

```
docker run -it \
  -d \
  --restart=always \
  --log-driver json-file \
  --log-opt max-size=100m \
  --log-opt max-file=2 \
  --name kibana \
  -p 5601:5601 \
  --net elastic \
  -v /etc/elk8.4.3/kibana/config:/usr/share/kibana/config \
  -v /etc/elk8.4.3/kibana/data:/usr/share/kibana/data \
  -v /etc/elk8.4.3/kibana/plugins:/usr/share/kibana/plugins \
  -v /etc/elk8.4.3/kibana/logs:/usr/share/kibana/logs \
  kibana:8.4.3

```

### 8. 配置索引生命周期

1. 配置策略 Index Lifecycle Policies

   ![img_v3_02ao_623ea326-f24e-4ebb-9b9c-d41cc894e9ag](https://panyuro.oss-cn-beijing.aliyuncs.com/img_v3_02ao_623ea326-f24e-4ebb-9b9c-d41cc894e9ag.jpg)

2. 创建索引模板

```
{
  "template": {
    "settings": {
      "index": {
        "lifecycle": {
          "name": "自定义的policy"
        },
        "routing": {
          "allocation": {
            "include": {
              "_tier_preference": "data_content"
            }
          }
        },
        "number_of_shards": "3",
        "number_of_replicas": "0"
      }
    },
    "aliases": {},
    "mappings": {}
  }
}
```

3. 给索引策略上加索引模板

   ![img_v3_02ao_a2e5305a-307f-4d19-a05a-17abb32f599g](https://panyuro.oss-cn-beijing.aliyuncs.com/img_v3_02ao_a2e5305a-307f-4d19-a05a-17abb32f599g.jpg)

## 2.4 安装logstash

### 1. 拉取镜像

```
docker pull logstash:8.4.3
```

### 2. 创建挂载目录并授权

```
mkdir /etc/elk8.4.3/logstash
sudo chown -R 1000:1000 /etc/elk8.4.3/logstash
```

### 3. 第一次执行docker启动脚本

```
docker run -it \
	-d \
	--name logstash \
	-p 9600:9600 \
	-p 5044:5044 \
	--net elastic \
	logstash:8.4.3
```

### 4. 将容器内的文件复制到主机上

```
docker cp logstash:/usr/share/logstash/config /etc/elk8.4.3/logstash/ 
docker cp logstash:/usr/share/logstash/pipeline /etc/elk8.4.3/logstash/ 
sudo cp /etc/elk8.4.3/elasticsearch/config/certs /etc/elk8.4.3/logstash/config/certs
```

### 5. 修改配置文件

- `/etc/elk8.4.3/logstash/config/logstash.yml`

```
http.host: "0.0.0.0"
xpack.monitoring.elasticsearch.hosts: [ "http://192.168.3.17:9200" ]
xpack.monitoring.elasticsearch.username: elastic
xpack.monitoring.elasticsearch.password: sith-mes
```

- `/etc/elk8.4.3/logstash/pipeline/logstash.conf`

```
input {
  beats {
    port => 5044
  }
}
 
filter {
      grok {  
      match => {"message" => "\[%{TIMESTAMP_ISO8601:time}\] \[%{LOGLEVEL:level}\] \[%{NOTSPACE:threadName}\] \[%{DATA:logger}\] \[%{DATA:hostname}\] \[%{DATA:ip}\] \[%{DATA:loginName}\] \[%{DATA:applicationName}\] \[%{DATA:executor}\] %{GREEDYDATA:restInfo}"}
    }

  date {
    match => [ "@timestamp", "yyyy-MM-dd HH:mm:ss Z" ]
  }
  mutate {
    remove_field => ["message", "host", "[event][original]"]
  }
}

output {
  elasticsearch {
    hosts => ["http://192.168.3.17:9200"]
    index => "log-%{+YYYY.MM.dd}"
    # ssl => true
    # ssl_certificate_verification => false
    # cacert => "/usr/share/logstash/config/certs/http_ca.crt"
    # ca_trusted_fingerprint => "第一次启动elasticsearch是保存的信息中查找"
    user => "elastic"
    password => "sith-mes"
  }
```

### 6. 删除容器

```
docker rm /logstash
```

### 7. 修改docker启动脚本，增加-v挂载目录

```
docker run -it \
  -d \
  --name logstash \
  -p 9600:9600 \
  -p 5044:5044 \
  --net elastic \
  -v /etc/elk8.4.3/logstash/config:/usr/share/logstash/config \
  -v /etc/elk8.4.3/logstash/pipeline:/usr/share/logstash/pipeline \
  logstash:8.4.3
```

## 2.5 安装filebeat

### 1. 拉取镜像

```
docker pull elastic/filebeat:8.4.3
```

### 2. 创建挂载目录并授权

```
mkdir /etc/elk8.4.3/filebeat
sudo chown -R 1000:1000 /etc/elk8.4.3/filebeat
```

### 3. 第一次执行docker启动脚本

```
docker run -it \
	-d \
	--name filebeat \
	--network host \
	-e TZ=Asia/Shanghai \
	elastic/filebeat:8.4.3 \
	filebeat -e  -c /usr/share/filebeat/filebeat.yml
```

### 4. 将容器内的文件复制到主机上

```
docker cp filebeat:/usr/share/filebeat/filebeat.yml /etc/elk8.4.3/filebeat/ 
docker cp filebeat:/usr/share/filebeat/data /etc/elk8.4.3/filebeat/ 
docker cp filebeat:/usr/share/filebeat/logs /etc/elk8.4.3/filebeat/ 
```

### 5. 修改配置文件

-  `/etc/elk8.4.3/filebeat/filebeat.yml`

```
filebeat.config:
  modules:
    path: ${path.config}/modules.d/*.yml
    reload.enabled: false

processors:
  - add_cloud_metadata: ~
  - add_docker_metadata: ~

output.logstash:
  enabled: true
  # The Logstash hosts
  hosts: ["192.168.3.17:5044"]
 
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /usr/share/filebeat/target/*/info.log  # 这个路径是需要收集的日志路径，是docker容器中的路径
  scan_frequency: 10s
  exclude_lines: ['HEAD']
  exclude_lines: ['HTTP/1.1']
  multiline.pattern: '^[[:space:]]+(at|\.{3})\b|Exception|捕获异常'
  multiline.negate: false
  multiline.match: after
```

### 6. 删除容器

```
docker rm /filebeat
```

### 7. 修改docker启动脚本，增加-v挂载目录

```
docker run -it \
  -d \
  --name filebeat \
  --network host \
  -e TZ=Asia/Shanghai \
  -v /home/sith/deploy/backend/logs:/usr/share/filebeat/target \
  -v /etc/elk8.4.3/filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml \
  -v /etc/elk8.4.3/filebeat/data:/usr/share/filebeat/data \
  -v /etc/elk8.4.3/filebeat/logs:/usr/share/filebeat/logs \
  elastic/filebeat:8.4.3 \
  filebeat -e  -c /usr/share/filebeat/filebeat.yml
```

⚠️： ES映射的data和log比较大，建议放在磁盘空间比较足的目录。



附：docker-compose.yml文件

```yml
networks:
  tms:
    external: true
services:
  elasticsearch:
    image: elasticsearch:8.4.3
    container_name: elasticsearch
    user: "root"  # 设置容器内用户为 root
    environment:
      - discovery.type=single-node
      - ELASTIC_PASSWORD=plm1234567890
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - 9200:9200
      - 9300:9300
    volumes:
      - ./elasticsearch/config:/usr/share/elasticsearch/config
      - ./elasticsearch/data:/usr/share/elasticsearch/data
      - ./elasticsearch/plugins:/usr/share/elasticsearch/plugins
      - ./elasticsearch/logs:/usr/share/elasticsearch/logs
    networks:
      - tms

  logstash:
    image: logstash:8.4.3
    container_name: logstash
    user: "root"  # 设置容器内用户为 root
    ports:
      - "5044:5044"
    volumes:
      - ./logstash/config:/usr/share/logstash/config
      - ./logstash/pipeline:/usr/share/logstash/pipeline
    restart: always
    networks:
      - tms

  kibana:
    image: kibana:8.4.3
    container_name: kibana
    user: "root"  # 设置容器内用户为 root
    ports:
      - 5601:5601
    depends_on:
      - elasticsearch
    volumes:
      - ./kibana/config:/usr/share/kibana/config
      - ./kibana/data:/usr/share/kibana/data
      - ./kibana/plugins:/usr/share/kibana/plugins
      - ./kibana/logs:/usr/share/kibana/logs
    networks:
      - tms

  filebeat:
    image: elastic/filebeat:8.4.3
    container_name: filebeat
    user: "root"  # 设置容器内用户为 root
    volumes:
      - /home/Z004NNRE/tms/deploy/backend/assistant-service/logs:/usr/share/filebeat/target/assistant-service
      - /home/Z004NNRE/tms/deploy/backend/auth-service/logs:/usr/share/filebeat/target/auth-service
      - /home/Z004NNRE/tms/deploy/backend/user-service/logs:/usr/share/filebeat/target/user-service
      - /home/Z004NNRE/tms/deploy/backend/tms-service/logs:/usr/share/filebeat/target/tms-service
      - /home/Z004NNRE/tms/deploy/backend/gateway-server/logs:/usr/share/filebeat/target/gateway-server
      - /home/Z004NNRE/tms/deploy/backend/eureka-server/logs:/usr/share/filebeat/target/eureka-server
      - ./filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml
      - ./filebeat/data:/usr/share/filebeat/data
      - ./filebeat/logs:/usr/share/filebeat/logs
    depends_on:
      - logstash
    networks:
      - tms

```





**注意：Docker中出现bash: vim: command not found解决方案**

解决这个问题其实非常简单：只需要在你的Docker容器内安装vim即可。具体步骤如下：

1. 首先打开你正在运行的docker容器：
   `docker exec -it [container_id] /bin/bash`

2. 进入到了docker环境后, 我们可以通过apt-get（如果您使用基于Debian或Ubuntu等Linux发行版）或yum（如果您使用基于CentOS等Linux发行版）来安装Vim：
   - `apt-get update && apt-get install vim -y   `Debian/Ubuntu等系统 
   - CentOS/RHEL等系统`yum update && yum install vim -y ` 

3. 以上步骤虽然能够解决问题，但每次创建新的Docker容器时，都需要手动执行这些步骤，显然不是一个理想的解决方案。更好的方法是在Dockerfile中添加安装vim编辑器的命令。这样，在构建docker镜像时就会自动安装vim。

   在你的Dockerfile中添加以下命令：

   ```cobol
   RUN apt-get update && apt-get install vim -y  # Debian/Ubuntu等系统
    
   RUN yum update && yum install vim -y  # CentOS/RHEL等系统
   ```
