---
title: Skywalking监控工具
date: 2025-03-12 19:28:06
tags:
categories: 应用监控工具
---

# 1. SkyWalking

## 1. 定义：

SkyWalking是一个开源的分布式追踪、应用性能监控（APM）和可观测性分析平台。 它主要功能有：分布式链路追踪、应用性能监控、数据库监控、服务依赖分析、告警等。

## 2. 主要功能

1. **分布式追踪**：

   - 追踪分布式系统中的请求流程，记录每个请求经过的服务和组件，以及请求在各个组件中的耗时情况。

   - 通过分析追踪数据，帮助开发者和运维人员了解系统中各个组件之间的调用关系和性能瓶颈，快速定位和解决问题。

     ![img_v3_02ka_9ddee659-f618-4d72-b0f4-a1a6a8841d3g](https://panyuro.oss-cn-beijing.aliyuncs.com/img_v3_02ka_9ddee659-f618-4d72-b0f4-a1a6a8841d3g.jpg)

   > 如上：可以根据traceID查询到这个请求的调用链路

2. **应用性能监控**：

   - 实时监控服务的各项性能指标，如响应时间、吞吐量（QPS）、错误率等。

   - 提供丰富的数据可视化功能，将监控数据以图表的形式展示，帮助用户更直观地了解系统的性能和运行情况。

     ![img_v3_02ka_699da6ea-8415-4ff6-beeb-11e40b89018g](https://panyuro.oss-cn-beijing.aliyuncs.com/img_v3_02ka_699da6ea-8415-4ff6-beeb-11e40b89018g.jpg)

3. **数据库性能监控**：

   - 监控数据库响应时间、访问成功率等

   - 监控数据库慢查询及耗时时间

     ![img_v3_02ka_e2a3bce4-7b37-493d-89b6-ea865350a09g](https://panyuro.oss-cn-beijing.aliyuncs.com/img_v3_02ka_e2a3bce4-7b37-493d-89b6-ea865350a09g.jpg)

4. **服务依赖分析**：

   - 分析系统中各个服务之间的依赖关系，包括调用关系和数据流向。

   - 通过可视化的方式展示服务之间的依赖关系，帮助开发者理解系统的架构和流程，优化系统设计。

     ![img_v3_02ka_308260ef-7a9d-4337-9cf8-0b4d8be004dg](https://panyuro.oss-cn-beijing.aliyuncs.com/img_v3_02ka_308260ef-7a9d-4337-9cf8-0b4d8be004dg.jpg)

5. **告警和报警**：

   - 支持基于性能指标的告警设置，当系统出现异常或性能下降时，及时发送告警通知。

   - 提供多种告警通知方式，如邮件、短信、Webhook等，帮助运维人员快速响应和解决问题。

     ![img_v3_02ka_fcaae416-148d-4040-89a0-24b7a4b5fb1g](https://panyuro.oss-cn-beijing.aliyuncs.com/img_v3_02ka_fcaae416-148d-4040-89a0-24b7a4b5fb1g.jpg)

> [具体告警配置](https://github.com/apache/skywalking/blob/v9.3.0/docs/en/setup/backend/backend-alarm.md)

## 3. Docker-compose 安装

我这里是持久化到elasticsearch， 修改了映射出来的端口为8088

```yml
services:
  elasticsearch:
    image: elasticsearch:8.4.3
    container_name: elasticsearch
    restart: always
    ports:
      - 9200:9200
    environment:
      - "TAKE_FILE_OWNERSHIP=true" #volumes 挂载权限 如果不想要挂载es文件改配置可以删除
      - "discovery.type=single-node" #单机模式启动
      - "TZ=Asia/Shanghai" # 设置时区
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m" # 设置jvm内存大小
    volumes:
      - ./elasticsearch/logs:/usr/share/elasticsearch/logs
      - ./elasticsearch/data:/usr/share/elasticsearch/data
      - ./elasticsearch/conf:/usr/share/elasticsearch/config
    ulimits:
      memlock:
        soft: -1
        hard: -1
  skywalking-oap-server:
    image: apache/skywalking-oap-server:9.3.0
    container_name: skywalking-oap-server
    depends_on:
      - elasticsearch
    links:
      - elasticsearch
    restart: always
    ports:
      - 11800:11800
      - 12800:12800
    environment:
      SW_STORAGE: elasticsearch
      SW_STORAGE_ES_CLUSTER_NODES: elasticsearch:9200
      TZ: Asia/Shanghai
    volumes:
     - ./oap/config:/skywalking/config
  skywalking-ui:
    image: apache/skywalking-ui:9.3.0
    container_name: skywalking-ui
    depends_on:
      - skywalking-oap-server
    links:
      - skywalking-oap-server
    restart: always
    ports:
      - 8088:8080
    environment:
      SW_OAP_ADDRESS: http://skywalking-oap-server:12800
      TZ: Asia/Shanghai
```

## 4. 集成到项目里

1. 项目日志支持打印traceId, 只需要引入sleuth

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-sleuth</artifactId>
   </dependency>
   ```

   注：如果使用了日志模版，则需要修改对应配置, 我的项目使用的是log4j2.xml

   ```
   <Console name="consoleAppender" target="SYSTEM_OUT">           
     <PatternLayout pattern="xxxx[%style{%X{X-B3-TraceId}}{green},%style{%X{X-B3-SpanId}}     {yellow},%X{X-B3-ParentSpanId}] xxxx"/>    
   </Console>
   ```

2. 到官网下载Java-agent, 按需修改config/agent.config的内容, 我这里没做修改，服务名都是后面通过VM option指定的

3. 项目运行的时候，加上VM option，就可以在界面看到啦！

   ```java
   -javaagent:D:\software\Skywalking\skywalking-java-agent\skywalking-agent/skywalking-agent.jar
   -Dskywalking.agent.service_name=sith-mes（可选）
   ```

   

**TIPS**： 

1. SkyWalking 不能检测服务是否存活，如果要检测服务是否存活，可以考虑使用Prometheus + Grafana+actuator
2. Zipkin也能实现链路追踪，且他更轻量，只是不能监控 CPU、内存、数据库性能、触发告警等
3. 可以通过nacos等来存储告警配置文件，使得skywalking实现告警规则动态刷新
