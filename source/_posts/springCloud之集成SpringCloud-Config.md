---
title: springCloud之集成SpringCloud Config
date: 2022-07-18 17:37:36
tags: 
- 分布式 
- 集中化配置
categories: springCloud
---

# 集中化配置

## 为什么要集中化配置？

- 微服务数量多，配置多
- 手工管理配置繁琐

## 配置分类

- 按照来源，分为：源代码，文件，数据库连接，服务调用等
- 按照配置环境，分为开发环境，测试环境，生产环境
- 集成阶段，编译，打包，运行

## 配置中心的要求

- 面向可配置的编码：避免硬编码，设置常量，运行时可读取
- 个理性：生产环境的配置和测试环境配置不一样
- 一致性：相同部署环境下，应该配置相同
- 集中化配置：分布式环境下，应用的配置应该具备可以管理性，提供远程管理的能力。



# SpringCloud Config

## springCloud config 是什么？

- Spring Cloud Config 是由 Spring Cloud 团队开发的项目，它可以为微服务架构中各个微服务提供集中化的外部配置支持。
- Spring Cloud Config 可以将各个微服务的配置文件集中存储在一个外部的存储仓库或系统（例如 Git 、SVN 等）中，对配置的统一管理，以支持各个微服务的运行。
- Spring Cloud Config 包含以下两个部分：
  - Config Server：也被称为分布式配置中心，它是一个独立运行的微服务应用，用来连接配置仓库并为客户端提供获取配置信息、加密信息和解密信息的访问接口。
  - Config Client：指的是微服务架构中的各个微服务，它们通过 Config Server 对配置进行管理，并从 Config Sever 中获取和加载配置信息。

## 搭建 Config 服务端

1. 搭建Eureka Client : [springCloud之Eureka搭建 - 楼上有只喵 (panyurou.github.io)](https://panyurou.github.io/2022/08/17/springCloud之Eureka搭建/)

2. github或者码云上创建一个仓库存储配置信息，github国内不是很稳定，我用了码云

   ![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5d7fi607gj21ja0u0q8a.jpg)

3. 在config-repo路径下，编写配置文件config-dev.yml

   ![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5d7h5hk5oj21h10u0n1z.jpg)

   

4. 引入依赖

   ```java
   implementation 'org.springframework.cloud:spring-cloud-config-server'
   ```

3. properties修改配置

   ```java
   server.port=8888
   spring.application.name: micro-weather-eureka-server-config
   eureka.client.serviceUrl.defaultZone: http://localhost:8761//eureka/
   
   # git 地址，用来存储配置文件
   spring.cloud.config.server.git.uri=https://gitee.com/panyuro/spring-cloud-microservice-config.git
   
   #存储配置文件的目录
   spring.cloud.config.server.git.searchPaths=config-repo
   ```

4. 启动类加入注解@EnableConfigServer

   ```java
   @SpringBootApplication
   @EnableDiscoveryClient
   @EnableConfigServer
   public class Application {
     public static void main(String[] args) {
       SpringApplication.run(Application.class, args);
     }
   }
   ```

5. 启动项目，访问[localhost:8888/config-dev.yml](http://localhost:8888/config-dev.yml)，我们可以看到8888微服务可以成功的读取到远端的配置文件。如下图：

   <img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h5d7ik0y9ij20qy0ag74v.jpg" style="zoom:50%;" />

访问[localhost:8888/config/dev/master](http://localhost:8888/config/dev/master)

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5d7jb4rhyj225k09m0vc.jpg)



## 搭建 Config客户端

1. 搭建Eureka Client : [springCloud之Eureka搭建 - 楼上有只喵 (panyurou.github.io)](https://panyurou.github.io/2022/08/17/springCloud之Eureka搭建/)

2. 引入依赖

   ```java
   	implementation 'org.springframework.cloud:spring-cloud-config-client'
   ```

   springcloud 版本较高，无法成功启动项目的时候，需要再引入starter-bootstrap

   ```java
   	implementation 'org.springframework.cloud:spring-cloud-starter-bootstrap'
   ```

3. properties修改配置

   ```java
   server.port=9091
   spring.application.name: micro-weather-config-client
   eureka.client.serviceUrl.defaultZone: http://localhost:8761//eureka/
   
   spring.cloud.config.profile=dev
   spring.cloud.config.uri=http://localhost:8888/
   spring.cloud.config.name=config
   ```

4. 编写测试代码

   ```java
   @RestController
   public class ConfigController {
   
     @Value("${config.info}")
     private String info;
   
     @Value("${config.version}")
     private String version;
   
     @GetMapping("/config")
     public String config(){
       return "info: "+info+" version: "+version;
     }
   }
   ```

5. 测试，访问[localhost:9091/config](http://localhost:9091/config)，可以查询到正确的配置

   <img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h5d7p3re81j20na08sgm3.jpg" style="zoom:50%;" />

   

# Config客户端之动态刷新

避免每次修改githib上的配置都需要重启客户端

## 手动版

优点：解决了重启 Config 客户端才能获取最新配置的问题，

缺点：只要配置仓库中的配置发生改变，就需要我们挨个向 Config 客户端手动发送 POST 请求，通知它们重新拉取配置。

那么有没有“一次通知，处处生效”的方式呢？答案是肯定的。Spring Cloud Config 配合 Bus 就可以实现配置的动态刷新。

