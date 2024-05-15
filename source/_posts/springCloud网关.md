---
title: springCloud网关
date: 2022-07-18 22:06:53
tags: 分布式
categories: 微服务
---

# 1.网关是什么

## 1.1 网关产生的原因？

1. 在微服务架构中，一个系统往往由多个微服务组成，而这些服务可能部署在不同机房、不同地区、不同域名下。这种情况下，客户端想要直接请求这些服务，就需要知道它们具体的地址信息，例如 IP 地址、端口号等。这种客户端直接请求服务的方式存在以下问题：

   - 当服务数量众多时，客户端需要维护大量的服务地址，这对于客户端来说，是非常繁琐复杂的。

   - 在某些场景下可能会存在跨域请求的问题。

   - 身份认证的难度大，每个微服务需要独立认证。

2. 在微服务架构中，每个服务都有对外提供服务的地址和端口，都有一些**共同的功能**，比如：**权限校验、日志记录、限流、熔断、监控**等等。如果任由每个服务独立对外提供接口。**协调管理这些接口**就会变得非常麻烦， 且会**造成代码的重复**。

## 1.2 网关是什么？

API 网关是一个**搭建在客户端和微服务之间的服务**。

- API 网关就像整个微服务系统的门面一样，是系统对外的唯一入口。有了它，客户端会先将请求发送到 API 网关，然后由 API 网关根据请求的标识信息将请求转发到微服务实例。

- 我们可以在 API 网关中处理一些非业务功能的逻辑，例如权限验证、监控、缓存、请求路由等。

  ![image-20230228232252488](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228232252488.png)

## 1.3 网关的优点和缺点

**优点：**

- 集合多个API
- 统一API入口，客户端通过 API 网关与微服务交互时，客户端只需要知道 API 网关地址即可，而不需要维护大量的服务地址，简化了客户端的开发。
- 客户端直接与 API 网关通信，能够减少客户端与各个服务的交互次数。
- API 网关还提供了安全、流控、过滤、缓存、计费以及监控等 API 管理功能。

**缺点：**

- 在架构上需要考虑更多的管理，比如哪些API放在哪个网关上
- 路由上需要进行统一的配置
- 可能引发单点故障，API是一个独立的入口，很容易出现高并发，设计不好，如果API网管不可用，整个系统不可用

## 1.4 API网关的实现方式

- Spring Cloud Gateway
- pring Cloud Netflix Zuul
- Nginx
- Traefik
- Kong: 底层也是基于ngin x

# 2 Zuul

Zuul 是Netflix开发的一款API网关组件。

优点：

1. 成熟稳定：Zuul 是 Netflix 开源的项目，已经在大规模生产环境中被验证过。
2. 社区支持：拥有成熟的社区，有大量的文档和案例可供参考。
3. 易于上手：相对较简单，易于配置和使用。

缺点：

1. 阻塞式 I/O：Zuul 使用阻塞式 I/O，每个请求都会占用一个线程，可能会导致线程资源耗尽。
2. 动态路由困难：Zuul 在运行时动态更新路由规则比较困难，需要重启服务。
3. 不支持响应式编程：Zuul 1.x 不支持响应式编程模型，无法充分利用异步和非阻塞的特性。

## 2.1 SpringCloud 集成Zuul

1. 搭建Eureka Client : [springCloud之Eureka搭建 - 楼上有只喵 (pyr9.github.io)](https://pyr9.github.io/2022/08/17/springCloud之Eureka搭建/)

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

![image-20230228232304736](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228232304736.png)



# 3 Spring Cloud Gateway

在Zulu2 中，zuul的技术方案一直在跳票，spring cloud 自己开发了一个网关Spring Cloud Gateway，用来取代Zuul网关。

- **响应式编程支持**：Spring Cloud Gateway 是基于 WebFlux 框架实现的，而 WebFlux 框架底层则使用了通信框架 Netty。这意味着它能够更高效地处理大量并发请求，而不会出现线程阻塞和资源浪费的问题。（Netty 使用了 Java NIO 提供的非阻塞 I/O 模型，能够处理大量并发的连接和请求，并且不会阻塞线程，提高了系统的并发处理能力和性能。）

- **Spring Cloud 生态集成**：作为 Spring Cloud 生态系统的一部分，Spring Cloud Gateway 与其他 Spring Cloud 组件（如 Eureka、Consul、Ribbon 等）有着良好的集成，可以方便地构建和管理微服务架构。

## 3.1 Spring Cloud Gateway 核心概念

- Route（路由）： 网关最基本的模块。它由一个 ID、一个目标 URI、一组断言（Predicate）和一组过滤器（Filter）组成。
- Predicate（断言）:路由转发的判断条件，我们可以通过 Predicate 对 HTTP 请求进行匹配，例如请求方式、请求路径、请求头、参数等，**如果请求与断言匹配成功，则将请求转发到相应的服务。**
- Filter（过滤器）:过滤器，使用过滤器，我们可以对请求进行拦截和修改，还可以使用它对上文的响应进行再处理。

**注意：其中 Route 和 Predicate 必须同时声明。**

使用了一个RouteLocatorBuilder的bean去创建路由，除了创建路由 RouteLocatorBuilder可以让你添加各种predicates和filters，predicates断言 的意思，顾名思义就是根据具体的请求的规则，由具体的route去处理，filters 是各种过滤器，用来对请求做各种判断和修改。

## 3.2 Spring Cloud Gateway 特性

- 基于 Spring Framework 5、Project Reactor 和 Spring Boot 2.0 构建。
- 能够在任意请求属性上匹配路由。
- predicates（断言） 和 filters（过滤器）是特定于路由的。
- 集成了 Hystrix 熔断器。
- 集成了 Spring Cloud DiscoveryClient（服务发现客户端）。
- 易于编写断言和过滤器。
- 能够限制请求频率。
- 能够重写请求路径。



## 3.3 Gateway 的工作流程

**核心逻辑：路由转发+执行过滤器**

1. 客户端将请求发送到 Spring Cloud Gateway 上。

2. Spring Cloud Gateway 通过 Gateway Handler Mapping 找到与请求相匹配的路由，将其发送给 Gateway Web Handler。

3. Gateway Web Handler 通过指定的过滤器链（Filter Chain），将请求转发到实际的服务节点中，执行业务逻辑返回响应结果。

4. 过滤器之间用虚线分开是因为过滤器可能会在转发请求之前（pre）或之后（post）执行业务逻辑。

5. 过滤器（Filter）可以在请求被转发到服务端前，对请求进行拦截和修改，例如参数校验、权限校验、流量监控、日志输出以及协议转换等。

6. 过滤器可以在响应返回客户端之前，对响应进行拦截和再处理，例如修改响应内容或响应头、日志输出、流量监控等。

7. 响应原路返回给客户端。

   ![image-20230228224346059](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228224346059.png)

## 3.4 Spring Cloud Gateway 动态路由

默认情况下，Spring Cloud Gateway 会根据服务注册中心（例如 Eureka Server）中维护的服务列表，以服务名（spring.application.name）作为路径创建动态路由进行转发，从而实现动态路由功能。

我们可以在配置文件中，将 Route 的 uri 地址修改为以下形式。

```java
lb://service-name
```

以上配置说明如下：

- lb：uri 的协议，表示开启 Spring Cloud Gateway 的负载均衡功能。
- service-name：服务名，Spring Cloud Gateway 会根据它获取到具体的微服务地址。

## 3.5 SpringCloud 使用GateWay

1. 搭建Eureka Client : [springCloud之Eureka搭建 - 楼上有只喵 (pyr9.github.io)](https://pyr9.github.io/2022/08/17/springCloud之Eureka搭建/)

2. 引入依赖

   ```java
   	implementation 'org.springframework.cloud:spring-cloud-starter-gateway'
   ```

3. 修改application.yml

   ```xml
   server:
     port: 9090
   spring:
     application:
       name: micro-weather-eureka-client-gateway
     cloud:
       gateway: #网关路由配置
         routes:
           #将 micro-service-cloud-provider-dept-8001 提供的服务隐藏起来，不暴露给客户端，只给客户端暴露 API 网关的地址 9527
           - id: micro_weather_eureka_client_path #路由 id,没有固定规则，但唯一，建议与服务名对应
             uri: http://localhost:9091         #匹配后提供服务的路由地址
             predicates:
               #以下是断言条件，必选全部符合条件
               - Path=/test/**               #断言，路径匹配 注意：Path 中 P 为大写
               - Method=GET #只能时 GET 请求时，才能访问
   eureka:
     client:
       fetch-registry: true
       register-with-eureka: true
       service-url:
         defaultZone: http://localhost:8761//eureka/
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

5. 测试

   可以看到根据路由匹配，通过http://localhost:9090/访问到了http://localhost:9091/的接口

   - 访问[localhost:9091/test/张三](http://localhost:9091/test/张三)

   ![image-20230228224437860](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228224437860.png)

   - 访问[localhost:9090/test/张三](http://localhost:9090/test/张三)

   ![image-20230228224445551](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228224445551.png)



**Spring Cloud Gateway 动态路由**

修改application.yml 的配置，使用注册中心中的微服务名创建动态路由进行转发

![image-20230228224403815](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228224403815.png)



# 4 zuul和spring cloudgateway比较

|              | Gateway              | Zuul1            | Zuul2        |
| ------------ | -------------------- | ---------------- | ------------ |
| 靠谱性       | spring cloud官方支持 | 曾经靠谱         | 专业放鸽子   |
| 性能         | Netty                | 同步阻塞，性能慢 | Netty        |
| RPS          | >32000               | 20000左右        | 25000左右    |
| spring cloud | 已整合               | 已整合           | 暂无整合计划 |
| 长链接       | 支持                 | 不支持           | 支持         |

