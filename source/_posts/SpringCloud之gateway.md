---
title: SpringCloud之gateway
date: 2022-08-22 23:15:11
tags:
-gateWay
-网关
categories: SpringCloud
---

# Spring Cloud Gateway

- Spring Cloud Gateway是Spring Cloud官方推出的第二代网关框架，用来取代Zuul网关。
-  Spring WebFlux(底层采用了高性能通信框架netty)，高并发和非阻塞通信就非常有优势
- 在Zulu2 中，zuul的技术方案一直在跳票，spring cloud 自己开发了一个网关替换zuu l
- Spring Cloud Gateway目标提供统一的路由方式，且基于filter链提供了网关基本功能，例如安全，监控/指标，限流 

网关作为流量的，在微服务系统中有着非常作用，网关常见的功能有路由转发、权限校验、限流控制等作用。 

使用了一个RouteLocatorBuilder的bean去创建路由，除了创建路由 RouteLocatorBuilder可以让你添加各种predicates和filters，predicates断言 的意思，顾名思义就是根据具体的请求的规则，由具体的route去处理，filters 是各种过滤器，用来对请求做各种判断和修改。



![image-20220822233547827](/Users/panyurou/Library/Application Support/typora-user-images/image-20220822233547827.png)







![image-20220822233648925](/Users/panyurou/Library/Application Support/typora-user-images/image-20220822233648925.png)





![image-20220822233821755](/Users/panyurou/Library/Application Support/typora-user-images/image-20220822233821755.png)







![image-20220822234201048](/Users/panyurou/Library/Application Support/typora-user-images/image-20220822234201048.png)



![image-20220822234227837](/Users/panyurou/Library/Application Support/typora-user-images/image-20220822234227837.png)



![image-20220822234337901](/Users/panyurou/Library/Application Support/typora-user-images/image-20220822234337901.png)

![image-20220822234316497](/Users/panyurou/Library/Application Support/typora-user-images/image-20220822234316497.png)
