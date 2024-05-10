---
title: springCloud之Eureka搭建
date: 2022-07-17 23:02:31
tags: 微服务
categories: 微服务
---

# Eureka是什么？

- Eureka是 Netflix 公司开发的一款开源的服务注册与发现组件。
- Spring Cloud 将 Eureka 与 Netflix 中的其他开源服务组件（例如 Ribbon、Feign 以及 Hystrix 等）一起整合进 Spring Cloud Netflix 模块中，整合后的组件全称为 Spring Cloud Netflix Eureka。
- Eureka 是 Spring Cloud Netflix 模块的子模块，它是 Spring Cloud 对 Netflix Eureka 的二次封装，主要负责 Spring Cloud 的服务注册与发现功能。

# Eureka 两大组件

Eureka 采用 CS（Client/Server，客户端/服务器） 架构，它包括以下两大组件：

- **Eureka Server**：Eureka 服务注册中心，主要用于提供服务注册功能。当微服务启动时，会将自己的服务注册到 Eureka Server。Eureka Server 维护了一个可用服务列表，存储了所有注册到 Eureka Server 的可用服务的信息，这些可用服务可以在 Eureka Server 的管理界面中直观看到。

- **Eureka Client**：Eureka 客户端，通常指的是微服务系统中各个微服务，主要用于和 Eureka Server 进行交互。在微服务应用启动后，Eureka Client 会向 Eureka Server 发送心跳（默认周期为 30 秒）。若 Eureka Server 在多个心跳周期内没有接收到某个 Eureka Client 的心跳，Eureka Server 将它从可用服务列表中移除（默认 90 秒



# Eureka 自我保护机制

## 原理

- 默认情况下,EurekaClient会定时向EurekaServer端发送心跳，如果EurekaServer在一定时间内没有收到EurekaClient发送的心跳，便会把该实例从注册服务列表中剔除（默认是90秒)
- 但是在短时间内丢失大量的实例心跳，这时候EurekaServer会开启自我保护机制，[Eureka](https://so.csdn.net/so/search?q=Eureka&spm=1001.2101.3001.7020)不会踢出该服务，而是会对该服务的信息进行保存。
- Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其他节点上 (即保证当前节点依然可用)
- 当网络稳定时，当前实例新的注册信息会被同步到其他节点中

## 产生原因

为了防止EurekaClient正常运行，但是与EurekaServer网络不通（延迟、拥堵、卡顿）的情况下，EurekaServer不会对EurekaClient服务进行剔除，因为微服务本身是建康的，此时不应该注销这个微服务。

## 何时使用

本地环境建议禁止自我保护，生产环境建议开启自我保护。



## eureka停止更新了

[Home · Netflix/eureka Wiki (github.com)](https://github.com/Netflix/eureka/wiki)

![image-20230228232003650](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228232003650.png)



# Eureka Server搭建

参考[GitHub - spring-cloud-samples/eureka](https://github.com/spring-cloud-samples/eureka)

- Step1: 新建一个springBoot项目

- Step2: 修改依赖

  ```java
  plugins{
  	id 'org.springframework.boot' version '2.6.1'
  	id 'io.spring.dependency-management' version '1.0.8.RELEASE'
  	id 'java'
  }
  
  ext {
  	set('springCloudVersion', "2021.0.0")
  	name = 'Eureka Server'
  	description = 'Eureka Server demo project'
  	version='0.0.1-SNAPSHOT'
  	sourceEncoding='UTF-8'
  }
  
  repositories {
  	mavenCentral()
  	maven { url 'https://repo.spring.io/release/' }
  	maven { url "https://repo.spring.io/libs-snapshot-local" }
  	maven { url "https://repo.spring.io/libs-milestone-local" }
  	maven { url "https://repo.spring.io/libs-release-local" }
  	maven { url "https://repo.springsource.org/plugins-release" }
  }
  
  dependencyManagement {
  	imports {
  		mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
  	}
  }
  
  dependencies {
  	implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-server'
  	testImplementation 'org.springframework.boot:spring-boot-starter-test'
  }
  ```

- Step3: application配置

  ```xml
  server.port: 8761
  
  eureka.instance.hostname: localhost
  #eureka可以同时作为服务端和客户端，这里禁用掉他客户端的能力
  eureka.client.registerWithEureka: false
  eureka.client.fetchRegistry: false
  eureka.client.serviceUrl.defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
  
  #关闭自我保护机制，以确保注册中心将不可用的实例及时正确剔除
  eureka.server.enable-self-preservation=false
  ```

- step4.启动类加上@EnableEurekaServer注解

  ```java
  package com.pyr.spring.cloud.weather;
  
  import org.springframework.boot.SpringApplication;
  import org.springframework.boot.autoconfigure.SpringBootApplication;
  import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
  
  @SpringBootApplication
  @EnableEurekaServer
  public class Application {
  
    public static void main(String[] args) {
      SpringApplication.run(Application.class, args);
    }
  
  }
  
  ```

- 启动成功后，访问[Eureka](http://localhost:8761/)可以看到如下页面，可以看出此时还没有实例注册到该服务上。

  ![image-20230228232017385](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228232017385.png)

# Eureka Client搭建

- Step1: 新建一个springBoot项目

- Step2: 修改依赖

  ```java
  plugins{
  	id 'org.springframework.boot' version '2.6.1'
  	id 'io.spring.dependency-management' version '1.0.8.RELEASE'
  	id 'java'
  }
  
  ext {
  	set('springCloudVersion', "2021.0.0")
  	name = 'Eureka Server'
  	description = 'Eureka Server demo project'
  	version='0.0.1-SNAPSHOT'
      
  	sourceEncoding='UTF-8'
  }
  
  repositories {
  	mavenCentral()
  	maven { url 'https://repo.spring.io/release/' }
  	maven { url "https://repo.spring.io/libs-snapshot-local" }
  	maven { url "https://repo.spring.io/libs-milestone-local" }
  	maven { url "https://repo.spring.io/libs-release-local" }
  	maven { url "https://repo.springsource.org/plugins-release" }
  }
  
  dependencyManagement {
  	imports {
  		mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
  	}
  }
  
  dependencies {
  	implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
  	implementation 'org.springframework.boot:spring-boot-starter-web'
  
  	testImplementation 'org.springframework.boot:spring-boot-starter-test'
  }
  ```

- Step3: application配置

  ```java
  spring.application.name: micro-weather-eureka-client
  eureka.client.serviceUrl.defaultZone: http://localhost:8761//eureka/
  ```

- step4.启动类加上@EnableDiscoveryClient注解

  ```java
  package com.pyr.spring.cloud.weather;
  
  import org.springframework.boot.SpringApplication;
  import org.springframework.boot.autoconfigure.SpringBootApplication;
  import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
  
  @SpringBootApplication
  @EnableDiscoveryClient
  public class Application {
    public static void main(String[] args) {
      SpringApplication.run(Application.class, args);
    }
  }
  ```

- 刷新

  ![image-20230228232035911](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228232035911.png)
