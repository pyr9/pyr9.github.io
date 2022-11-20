---
title: SpringCloud集成consul
date: 2022-07-22 24:11:59
tags:
- 分布式
- 注册中心
- consul
categories: 微服务
---

# Consul

- Consul 是一套开源的分布式服务发现和配置管理系统，由HashCorp公司用Go语言开发
- 它提供了几个关键功能：
  - 服务发现：Consul client 可以提供服务，例如api或mysql，也可以使用Consul client来发现指定服务的提供者。 使用DNS或HTTP，应用程序可以轻松找到他们所依赖的服务。
  - 健康检查：Consul client 可以提供任何数量的健康检查，或者与给定的服务（“Web服务器是否返回200 OK”），或与本地节点（“内存利用率是否低于90％”）相关联。 可以使用此信息来监控集群运行状况，服务发现组件使用此信息将流量从有问题的主机中移除出去。
  - KV Store：应用程序可以使用Consul的分层键/值存储，包括动态配置，功能标记，协调，leader选举等等。 简单的HTTP API使其易于使用。
  - 多数据中心：Consul支持多个数据中心。 这意味着Consul的用户不必担心构建额外的抽象层以扩展到多个区域。

- 使用场景

  Consul的应用场景包括服务发现、服务隔离、服务配置：

  - 服务发现场景中consul作为注册中心，服务地址被注册到consul中以后，可以使用consul提供的dns、http接口查询，consul支持health check。

  - 服务隔离场景中consul支持以服务为单位设置访问策略，能同时支持经典的平台和新兴的平台，支持tls证书分发，service-to-service加密。

  - 服务配置场景中consul提供key-value数据存储功能，并且能将变动迅速地通知出去，通过工具consul-template可以更方便地实时渲染配置文件。

    

    

#  SpringCloud集成consul

[Spring Cloud Consul 中文文档 参考手册 中文版](https://www.springcloud.cc/spring-cloud-consul.html)

## 1. consul 安装和启动

参考[Install Consul | Consul - HashiCorp Learn](https://learn.hashicorp.com/tutorials/consul/get-started-install)

```
➜  ~ consul agent -dev
```

访问http://localhost:8500/

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5fjegc3rcj21xz0u0ad8.jpg)



## 2. 引入依赖

```xml
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
//	consul 依赖
	implementation 'org.springframework.cloud:spring-cloud-starter-consul-discovery'
	implementation 'org.springframework.boot:spring-boot-starter-web'
}
```



## 3.  修改配置文件

```xml
server.port=9091

spring.application.name= micro-weather-consul
spring.cloud.consul.host= localhost
spring.cloud.consul.port= 8500
spring.cloud.consul.discovery.service-name= ${spring.application.name}
# 打开心跳机制
spring.cloud.consul.discovery.heartbeat.enabled= true
```



### 3. 修改启动类

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



## 4.测试

再次访问http://localhost:8500/，可以看到micro-weather-consul服务已经注册成功

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h5fk8xt9m6j22190u0dj3.jpg)

