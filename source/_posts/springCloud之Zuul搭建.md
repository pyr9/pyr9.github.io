---
title: springCloud之集成Zuul
date: 2022-08-18 14:06:53
tags: 分布式
categories: springCloud
---

# API 网关

## API网关是什么？

在微服务架构中，每个微服务通常会以RESTFUL API的形式对外提供服务，API网关就是集合多个API，统一API入口，

## 优点和缺点

#### 好处

- 集合多个API
- 统一API入口
- 为微服务提供额外的安全层
- 支持混合通信协议

#### 坏处

- 在架构上需要考虑更多的管理，比如哪些API放在哪个网关上
- 路由上需要进行统一的配置
- 可能引发单点故障，API是一个独立的入口，很容易出现高并发，设计不好，如果API网管不可用，整个系统不可用

## API网关的实现

- Nginx
- Zuul
- Kong: 底层也是基于ngin x

# Zuul 是什么？

- Zuul 是Netflix开发的一款API网关组件，对请求提供路由及过滤功能。 

## SpringCloud 集成Zuul

1. 搭建Eureka Client : [springCloud之Eureka搭建 - 楼上有只喵 (panyurou.github.io)](https://panyurou.github.io/2022/08/17/springCloud之Eureka搭建/)

2. 引入依赖

```java
	implementation 'org.springframework.cloud:spring-cloud-starter-netflix-zuul'
```

**注：springCloud 版本过高，会存在zuul无法引入的问题，所以本地，将SpringCloud版本降低了，相应的SpringBoot版本也降低了**

```java
plugins{
	id 'org.springframework.boot' version '2.2.0.RELEASE'
}
ext {
	set('springCloudVersion', "Hoxton.SR12")
}
```



3. properties修改配置

```xml
server.port=9090
spring.application.name: micro-weather-eureka-client-zuul
eureka.client.serviceUrl.defaultZone: http://localhost:8761//eureka/

# 可配置多对路由
zuul.routes.demo.path: /demo/**
zuul.routes.demo.serviceId: micro-weather-eureka-client

#zuul.routes.city.path: /city/**
#zuul.routes.city.serviceId: msa-weather-city-eureka

#zuul.routes.data.path: /data/**
#zuul.routes.data.serviceId: msa-weather-data-eureka
```



4. 启动类加入注解@EnableZuulProxy

```java
package com.pyr.spring.cloud.weather;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@SpringBootApplication
@EnableDiscoveryClient
@EnableZuulProxy
public class Application {
  public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
  }
}
```

5. 访问[localhost:9090/demo/hello](http://localhost:9090/demo/hello)，得到micro-weather-eureka-client 服务的数据

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5azw9rud2j20l208o3yu.jpg)







