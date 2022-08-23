---
title: springCloud之集成Zuul
date: 2022-07-18 22:06:53
tags: 分布式
categories: springCloud
---

# API 网关

## API 网关产生的原因？

在微服务架构中，一个系统往往由多个微服务组成，而这些服务可能部署在不同机房、不同地区、不同域名下。这种情况下，客户端想要直接请求这些服务，就需要知道它们具体的地址信息，例如 IP 地址、端口号等。

这种客户端直接请求服务的方式存在以下问题：

- 当服务数量众多时，客户端需要维护大量的服务地址，这对于客户端来说，是非常繁琐复杂的。
- 在某些场景下可能会存在跨域请求的问题。
- 身份认证的难度大，每个微服务需要独立认证。

## API网关是什么？

- API 网关是一个搭建在客户端和微服务之间的服务，我们可以在 API 网关中处理一些非业务功能的逻辑，例如权限验证、监控、缓存、请求路由等。

- API 网关就像整个微服务系统的门面一样，是系统对外的唯一入口。有了它，客户端会先将请求发送到 API 网关，然后由 API 网关根据请求的标识信息将请求转发到微服务实例。

- 在微服务架构中，每个微服务通常会以RESTFUL API的形式对外提供服务，API网关就是集合多个API，统一API入口。

  ![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5gipv0ig0j21i40n8abz.jpg)

## 优点和缺点

#### 好处

- 集合多个API
- 统一API入口，客户端通过 API 网关与微服务交互时，客户端只需要知道 API 网关地址即可，而不需要维护大量的服务地址，简化了客户端的开发。
- 客户端直接与 API 网关通信，能够减少客户端与各个服务的交互次数。
- API 网关还提供了安全、流控、过滤、缓存、计费以及监控等 API 管理功能。

#### 坏处

- 在架构上需要考虑更多的管理，比如哪些API放在哪个网关上
- 路由上需要进行统一的配置
- 可能引发单点故障，API是一个独立的入口，很容易出现高并发，设计不好，如果API网管不可用，整个系统不可用

## API网关的实现

- Spring Cloud Gateway
- pring Cloud Netflix Zuul
- Nginx
- Traefik
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







