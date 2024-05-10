---
title: SpringCloud集成Ribbon
date: 2022-07-22 15:12:41
tags:
- 分布式
- 负载均衡
- Ribbon
categories: 微服务
---

# 负载均衡（load balance）

- 简单来说负载均衡就是将客户的请求平摊到多个服务上，从而达到系统的高可用，常用的负载均衡方式有Nginx，F5
- 常见的负载均衡方式有两种：
  - 服务端负载均衡
  - 客户端负载均衡

## 服务端负载均衡

### 工作原理

服务端负载均衡是最常见的负载均衡方式，其工作原理如下图。

![image-20230228225113562](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228225113562.png)

- 在客户端和服务端之间建立一个独立的负载均衡服务器，该服务器既可以是硬件设备（例如 F5），也可以是软件（例如 Nginx）
- 这个负载均衡服务器维护了一份可用服务端清单，然后通过心跳机制来删除故障的服务端节点，以保证清单中的所有服务节点都是可以正常访问的。
- 当客户端发送请求时，该请求不会直接发送到服务端进行处理，而是全部交给负载均衡服务器，由负载均衡服务器按照某种算法（例如轮询、随机等），从其维护的可用服务清单中选择一个服务端，然后进行转发。



### 服务端负载均衡的特点

服务端负载均衡具有以下特点：

- 需要建立一个独立的负载均衡服务器。
- 负载均衡是在客户端发送请求后进行的，因此客户端并不知道到底是哪个服务端提供的服务。
- 可用服务端清单存储在负载均衡服务器上。



## 客户端负载均衡

相较于服务端负载均衡，客户端服务是将负载均衡逻辑以代码的形式封装到客户端上

### 工作原理

客户端负载均衡的工作原理如下图。

![image-20230228225122260](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228225122260.png)

- 客户端通过服务注册中心（例如 Eureka Server）获取到一份服务端提供的可用服务清单。
- 有了服务清单后，负载均衡器会在客户端发送请求前通过负载均衡算法（如简单轮训，随机连接等）选择一个服务端实例再进行访问
- 客户端负载均衡也需要心跳机制去维护服务端清单的有效性，这个过程需要配合服务注册中心一起完成。



### 客户端负载均衡的特点：

- 负载均衡器位于客户端，不需要单独搭建一个负载均衡服务器。

- 负载均衡是在客户端发送请求前进行的，因此客户端清楚地知道是哪个服务端提供的服务。

- 客户端都维护了一份可用服务清单，而这份清单都是从服务注册中心获取的。

  

## Ribbon和Nginx实现负载均衡的区别

- Ribbon 就是一个基于 HTTP 和 TCP 的客户端负载均衡器，当我们将 Ribbon 和 Eureka 一起使用时，Ribbon 会从 Eureka Server（服务注册中心）中获取服务端列表，然后通过负载均衡策略（如简单轮训，随机连接等），从服务端列表选择一个服务实例进行访问，从而达到负载均衡的目的。
- 而Nginx是服务端负载均衡，客户端的所有请求会交给Nginx，然后Nginx实现转发请求



# Ribbon

- ribbon是Netflix开发的一款负载均衡的服务调用组件，具有多种负载均衡调用策略； 
- Ribbon 是 Spring Cloud Netflix 模块的子模块，它是 Spring Cloud 对 Netflix Ribbon 的二次封装。通过它，我们可以将面向服务的 RestTemplate请求转换为客户端负载均衡的服务调用。
- 简单来说，rubbon = 负载均衡+ restTemplate



# Ribbon 实现服务调用

1. 搭建Eureka Client : [springCloud之Eureka搭建 - 楼上有只喵 (pyr9.github.io)](https://pyr9.github.io/2022/08/17/springCloud之Eureka搭建/)

2. 引入依赖

   ```xml
   	implementation 'org.springframework.cloud:spring-cloud-starter-netflix-ribbon'
   ```

3. 修改application.properties

   ```xml
   server.port=8085
   spring.application.name: micro-weather-eureka-client-ribbon
   eureka.client.serviceUrl.defaultZone: http://localhost:8761//eureka/
   ```

4. 修改启动类

   ```java
   @SpringBootApplication
   @EnableDiscoveryClient
   public class Application {
   
     public static void main(String[] args) {
       SpringApplication.run(Application.class, args);
     }
   
   }
   ```

   

5. 创建一个名为 RestConfiguration 的配置类，将 RestTemplate 注入到容器中，代码如下。

   ```java
   package com.pyr.spring.cloud.weather.config;
   
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.web.client.RestTemplateBuilder;
   import org.springframework.cloud.client.loadbalancer.LoadBalanced;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.web.client.RestTemplate;
   
   @Configuration
   public class RestConfiguration {
   
     @Autowired
     private RestTemplateBuilder restTemplateBuilder;
   
     @Bean
     @LoadBalanced
     public RestTemplate restTemplate() {
       return restTemplateBuilder.build();
     }
   }
   ```

5. 创建一个CityController来实现通过restTemplate对服务的调用

   ```java
   package com.pyr.spring.cloud.weather.controller;
   
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   import org.springframework.web.client.RestTemplate;
   
   import java.util.Objects;
   
   @RestController
   public class CityController {
   
     @Autowired
     private RestTemplate restTemplate;
   
     //面向微服务编程，即通过微服务的名称来获取调用地址
     private static final String REST_URL_PREFIX = "http://micro-weather-city-eureka";
   
     @RequestMapping("/cities")
     public String listCity(){
       return restTemplate.getForObject(REST_URL_PREFIX + "/cities", String.class);
     }
   }
   ```

6. 访问[localhost:8085/cities，可以获取到指定服务接口的返回

   ![image-20230228225138488](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228225138488.png)



# Ribbon 实现负载均衡

java -jar msa-weather-city-eureka-0.0.1-SNAPSHOT.jar --server.port=8084

java -jar msa-weather-city-eureka-0.0.1-SNAPSHOT.jar --server.port=8083

java -jar msa-weather-city-eureka-0.0.1-SNAPSHOT.jar --server.port=8086

![image-20230228233156385](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228233156385.png)

再次访问[localhost:8085/cities](http://localhost:8085/cities)
