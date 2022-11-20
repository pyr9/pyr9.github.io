---
title: springCloud之Config+Bus
date: 2022-07-23 23:11:50
tags:
categories: 微服务
---

# 消息总线

- 在微服务架构的系统中，通常会使用**轻量级的消息代理**来构建一个公用的**消息主题**，并让系统中的所有实例都连接上来。由于该**主题的消息会被所有实例监听和消费**，所以称之为消息总线。

- 在总线上的各个实例，都可以方便的广播一些让其他连接在给主题实例上都知道的消息。

  

# Spring Cloud Bus 

- Spring Cloud Bus 又被称为消息总线，它能够通过轻量级的消息代理（例如 RabbitMQ、Kafka 等）将微服务架构中的各个服务连接起来，实现广播状态更改、事件推送等功能，还可以实现微服务之间的通信功能。

- 目前 Spring Cloud Bus 支持两种消息代理：RabbitMQ 和 Kafka。

## Spring Cloud Bus 的基本原理

- Spring Cloud Bus 会使用一个轻量级的消息代理来构建一个公共的消息主题 Topic（默认为“springCloudBus”），**这个 Topic 中的消息会被所有服务实例监听和消费**。
- 当其中的一个服务刷新数据时，Spring Cloud Bus 会把信息保存到 Topic 中，这样监听这个 Topic 的服务就收到消息并自动消费。



## Spring Cloud Bus 动态刷新配置的原理

- 当 Git 仓库中的配置发生了改变，我们只需要向某一个服务（既可以是 Config 服务端，也可以是 Config 客户端）发送一个 POST 请求，Spring Cloud Bus 就可以通过消息代理通知其他服务重新拉取最新配置，以实现配置的动态刷新。

- 利用 Spring Cloud Bus 的特殊机制可以实现很多功能，其中配合 Spring Cloud Config 实现配置的动态刷新就是最典型的应用场景之一。



## Spring Cloud Bus 整合Spring Cloud Config实现配置的动态刷新

1. 当 Git 仓库中的配置发生改变后，运维人员向 Config 服务端发送一个 POST 请求，请求路径为“/actuator/refresh”。
2. Config 服务端接收到请求后，会将该请求转发给服务总线 Spring Cloud Bus。
3. Spring Cloud Bus 接到消息后，会通知给所有 Config 客户端。
4. Config 客户端接收到通知，请求 Config 服务端拉取最新配置。
5. 所有 Config 客户端都获取到最新的配置。
