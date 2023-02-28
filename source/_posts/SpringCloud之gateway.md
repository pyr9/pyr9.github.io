---
title: SpringCloud之gateway
date: 2022-07-22 23:15:11
tags:
- gateWay
- 网关
categories: 微服务
---

# Spring Cloud Gateway

- Spring Cloud Gateway是Spring Cloud官方推出的第二代网关框架，用来取代Zuul网关。
-  Spring Cloud Gateway 是基于 WebFlux 框架实现的，而 WebFlux 框架底层则使用了高性能的 Reactor 模式通信框架 Netty。高并发和非阻塞通信就非常有优势
- 在Zulu2 中，zuul的技术方案一直在跳票，spring cloud 自己开发了一个网关替换zuu l
- Spring Cloud Gateway目标提供统一的路由方式，且基于filter链提供了网关基本功能，例如安全，监控/指标，限流 



## Spring Cloud Gateway 核心概念

- Route（路由）： 网关最基本的模块。它由一个 ID、一个目标 URI、一组断言（Predicate）和一组过滤器（Filter）组成。
- Predicate（断言）:路由转发的判断条件，我们可以通过 Predicate 对 HTTP 请求进行匹配，例如请求方式、请求路径、请求头、参数等，**如果请求与断言匹配成功，则将请求转发到相应的服务。**
- Filter（过滤器）:过滤器，使用过滤器，我们可以对请求进行拦截和修改，还可以使用它对上文的响应进行再处理。

**注意：其中 Route 和 Predicate 必须同时声明。**

使用了一个RouteLocatorBuilder的bean去创建路由，除了创建路由 RouteLocatorBuilder可以让你添加各种predicates和filters，predicates断言 的意思，顾名思义就是根据具体的请求的规则，由具体的route去处理，filters 是各种过滤器，用来对请求做各种判断和修改。



## Spring Cloud Gateway 特性

- 基于 Spring Framework 5、Project Reactor 和 Spring Boot 2.0 构建。
- 能够在任意请求属性上匹配路由。
- predicates（断言） 和 filters（过滤器）是特定于路由的。
- 集成了 Hystrix 熔断器。
- 集成了 Spring Cloud DiscoveryClient（服务发现客户端）。
- 易于编写断言和过滤器。
- 能够限制请求频率。
- 能够重写请求路径。



## Gateway 的工作流程

**核心逻辑：路由转发+执行过滤器**

1. 客户端将请求发送到 Spring Cloud Gateway 上。

2. Spring Cloud Gateway 通过 Gateway Handler Mapping 找到与请求相匹配的路由，将其发送给 Gateway Web Handler。

3. Gateway Web Handler 通过指定的过滤器链（Filter Chain），将请求转发到实际的服务节点中，执行业务逻辑返回响应结果。

4. 过滤器之间用虚线分开是因为过滤器可能会在转发请求之前（pre）或之后（post）执行业务逻辑。

5. 过滤器（Filter）可以在请求被转发到服务端前，对请求进行拦截和修改，例如参数校验、权限校验、流量监控、日志输出以及协议转换等。

6. 过滤器可以在响应返回客户端之前，对响应进行拦截和再处理，例如修改响应内容或响应头、日志输出、流量监控等。

7. 响应原路返回给客户端。

   ![image-20230228224346059](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228224346059.png)

## Spring Cloud Gateway 动态路由

默认情况下，Spring Cloud Gateway 会根据服务注册中心（例如 Eureka Server）中维护的服务列表，以服务名（spring.application.name）作为路径创建动态路由进行转发，从而实现动态路由功能。

我们可以在配置文件中，将 Route 的 uri 地址修改为以下形式。

```java
lb://service-name
```

以上配置说明如下：

- lb：uri 的协议，表示开启 Spring Cloud Gateway 的负载均衡功能。
- service-name：服务名，Spring Cloud Gateway 会根据它获取到具体的微服务地址。



# SpringCloud 使用GateWay

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



