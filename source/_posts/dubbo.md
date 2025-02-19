---
title: dubbo3.0
date: 2024-02-13 10:33:59
tags:
categories: 中间件
---

# 1. Dubbo和Feign的区别和联系

# 1. 选择建议

- 如果你的系统是 **中小型微服务架构**，更注重开发效率和易用性，且主要使用 HTTP 协议进行服务间通信，Feign 是一个更简单、更轻量级的选择。
- 如果你的系统是 **大规模分布式架构**，对性能要求较高，并且需要强大的服务治理能力（负载均衡、服务降级、限流、路由规则），建议使用 **Dubbo**。
- 如果你的项目已经使用了 Nacos、Sentinel、Seata 等阿里巴巴开源组件，Dubbo 的集成会更加顺畅。

# 2. 异同点

## 1. 相同点

1. 都是 **RPC 框架**
   - Dubbo 和 Feign 都是用来解决微服务之间的远程调用问题。
   - 它们的核心目标都是让开发者像调用本地方法一样调用远程服务。

2. 都**支持服务发现**
   - Dubbo 和 Feign 都可以与服务注册中心（如 ZooKeeper、Eureka、Nacos 等）集成，实现服务的动态发现和路由。

3. 都**支持负载均衡**
   - Dubbo 和 Feign 都提供了负载均衡的功能，能够将请求分发到多个服务实例。

4. 都**支持声明式编程**
   - Dubbo 和 Feign 都可以通过接口定义服务，减少手动编写代码的工作量。
   - Feign 的声明式风格更加简洁，而 Dubbo 的配置相对复杂但功能更强大。

5. 都**适合微服务架构**
   - Dubbo 和 Feign 都是为微服务架构设计的工具，能够帮助开发者构建分布式系统。

## 2. 区别

1. 使用的通信协议不同
   - dubbo默认使用自定义的二进制协议（Dubbo 协议），也可以选择其他协议（如 HTTP、gRPC 等）。Dubbo 协议底层基于 TCP 进行通信，性能较高，延迟较低。
   - Feign使用标准的 HTTP/HTTPS 协议进行通信。依赖 RESTful 风格的接口设计。
2. 序列化方式不同
   - Dubbo支持多种序列化方式，如 Hessian、Protobuf、JSON 等，默认使用高效二进制序列化（如 Hessian2）。适合对性能要求较高的场景。
   - Feign支持多种序列化方式，包括 JSON、XML、Form 表单等，默认使用 JSON 序列化，虽然简单易用，但性能不如二进制序列化。

3. 对注册中心依赖程度不同
   - Dubbo强依赖注册中心（如 ZooKeeper、Nacos、Consul 等）来管理服务的地址信息，客户端通过注册中心获取服务提供者的地址列表，从而完成服务调用。
   - Feign可以与 Eureka、Consul 等服务注册中心结合使用，但本身不强制依赖注册中心，当服务地址固定或通过配置文件指定时，Feign 也可以独立工作。

4. 实现负载均衡的方式不同
   - Dubbo内置了多种负载均衡策略（如随机、轮询、一致性哈希等）。
   - Feign依赖 Spring Cloud LoadBalancer/Ribbon（Spring Cloud 的负载均衡组件）实现负载均衡。

5. 支持容错机制方式不同
   - Dubbo提供多种容错策略
   - Feign集成 Hystrix 实现熔断和降级

# 4. dubbo使用

实现 UserService（服务提供者）

在服务提供者模块中实现 `UserService` 接口：

```java
package com.example.service;

import com.example.api.UserService;
import org.apache.dubbo.config.annotation.DubboService;

@DubboService(version = "1.0.0") // 声明这是一个 Dubbo 服务
public class UserServiceImpl implements UserService {

    @Override
    public String sayHello(String name) {
        return "Hello, " + name + "!";
    }
}
```

调用 UserService（服务消费者）

在服务消费者模块中通过 `@DubboReference` 注解引用远程服务：

```java
package com.example.consumer;

import com.example.api.UserService;
import org.apache.dubbo.config.annotation.DubboReference;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @DubboReference(version = "1.0.0") // 引用远程服务
    private UserService userService;

    @GetMapping("/hello")
    public String hello(@RequestParam String name) {
        return userService.sayHello(name); // 远程调用 sayHello 方法
    }
}
```

