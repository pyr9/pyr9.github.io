---
title: Actuator
date: 2025-03-13 21:50:53
tags:
categories:
---

# 1. Actuator 是什么？

Spring Boot Actuator 模块提供了生产级别的功能，比如健康检查，审计，指标收集，HTTP 跟踪等，帮助我们监控和管理Spring Boot 应用，上述的功能都可以通过HTTP 和 JMX 访问。

Actuator 也可以和一些外部的应用监控系统整合（Prometheus等）

# 2. 集成Springboot

1. 引入依赖

   ```xml
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-actuator</artifactId>
           </dependency>
   ```

   

2. 修改配置（按需）

   ```yml
   #开启actuator全面监控
   management:
     endpoint:
       health:
         show-details: always
   ```

3. 访问对应服务的 `/actuator/`

   ![image-20250313215237665](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250313215237665.png)

   ![](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250313215237665-20250313215311616-20250313215503904.png)
