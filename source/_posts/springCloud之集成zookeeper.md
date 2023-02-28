---
title: springCloud之集成zookeeper
date: 2022-07-22 20:45:26
tags:
- 分布式
- 注册中心
- zookeeper
categories: 微服务
---

# zookeeper

- zookeeper基本概念：[zookeeper特性与节点介绍 - 楼上有只喵 (pyr9.github.io)](https://pyr9.github.io/2021/09/22/zookeeper特性与节点介绍/)
- zookeeper 是一个分布式协调服务，可以用来实现注册中心功能，类似eureka
- 服务节点是临时结点



# springCloud之集成zookeeper

## 准备工作

启动zookeeper

## 1.引入依赖

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
	implementation 'org.springframework.cloud:spring-cloud-starter-zookeeper-discovery'
	implementation 'org.springframework.boot:spring-boot-starter-web'
}

test{
	useJUnitPlatform()
}
```

## 2. 修改配置文件

- 配置服务的名称和zookeeper的地址

```xml
server.port=9091

spring.application.name: micro-weather-zookeeper
spring.cloud.zookeeper.connect-string: localhost:2181
```

## 3. 修改启动类

- 加上@EnableDiscoveryClient注解

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



## 4.进入zookeeper客户端，查看已经注册的服务

可以看到 micro-weather-zookeeper服务已经存在，SpringCloud以[Zookeeper]为注册中心整合成功

![image-20230228233716687](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228233716687.png)
