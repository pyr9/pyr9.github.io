---
title: springCloud之集成Feign
date: 2022-07-18 19:13:24
tags: 分布式
categories: springCloud
---

# Feign是什么？

- Feign是Netflix开发的一款声明式服务调用组件
- Feign可以帮助我们更快捷、优雅地调用HTTP API。
- Feign 是 Spring Cloud Netflix 模块的子模块，它是 Spring Cloud 对 Netflix Feign 的二次封装，主要负责 Spring Cloud 的服务调用功能。

# SpringCloud 集成Feign

1. 搭建Eureka Client : [springCloud之Eureka搭建 - 楼上有只喵 (panyurou.github.io)](https://panyurou.github.io/2022/08/17/springCloud之Eureka搭建/)

2. 引入依赖

   ```java
   implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'
   ```

3. 修改application.properties

   ```xml
   server.port=8085
   spring.application.name: micro-weather-eureka-client-feign
   eureka.client.serviceUrl.defaultZone: http://localhost:8761//eureka/
   # 连接的超时时间
   feign.client.config.feignName.connectTimeout: 5000
   # 读数据的超时时间
   feign.client.config.feignName.readTimeout: 5000
   ```

   

4. 创建一个接口，用FeignClient注解，指明需要调用的服务名和rest接口

   ```java
   @FeignClient("micro-weather-city-eureka")
   public interface CityClient {
     @RequestMapping("/cities")
     String listCity();
   }
   ```

5. 修改启动类

   ```java
   package com.pyr.spring.cloud.weather;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
   import org.springframework.cloud.openfeign.EnableFeignClients;
   
   @SpringBootApplication
   @EnableDiscoveryClient
   @EnableFeignClients
   public class Application {
   
     public static void main(String[] args) {
       SpringApplication.run(Application.class, args);
     }
   }
   

6. 创建一个RestClientController来实现对Feign客户端的调用

   ```java
   package com.pyr.spring.cloud.weather.controller;
   
   import com.pyr.spring.cloud.weather.service.CityClient;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   @RestController
   public class CityController {
     @Autowired
     private CityClient cityClient;
   
     @RequestMapping("/cities")
     public String listCity(){
       return cityClient.listCity();
     }
   }
   ```

7. 查看euraka服务端，可以看到该客户端注册成功，访问[localhost:8085/cities，可以获取到指定服务接口的返回

   ![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5ap6lysrwj21ek0u00xb.jpg)

   ![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5ap87sv66j228m0k4zw9.jpg)
