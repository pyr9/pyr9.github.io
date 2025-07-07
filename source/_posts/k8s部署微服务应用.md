---
title: k8s部署微服务应用
date: 2025-07-05 16:06:49
tags:
categories: Kubernetes
---

# 1.  K8s部署Java项目的步骤

## 1. 构建镜像

### 1. 增加Dockerfile

```yml
# 使用官方的 OpenJDK 镜像作为基础镜像
FROM docker.xuanyuan.me/openjdk:8-jdk-alpine

# 设置工作目录
WORKDIR /app

# 复制本地的 JAR 包到容器中
COPY ./build/libs/micro-weather-weather-eureka-server.jar eureka-server.jar

EXPOSE 8761

# 启动应用
ENTRYPOINT ["java", "-jar", "eureka-server.jar"]
```

### 2. 构建镜像

```shell
#!/bin/bash

# 或者使用 echo $(pwd)
echo "当前工作目录是: $(pwd)"

docker build  -t eureka-server:02 .

docker images
```

## 2. **创建Deployment**

### 1. 增加配置文件

eureka-app-deployment.yaml

```yml
apiVersion: apps/v1                 # 使用的API版本，apps/v1适用于Deployment等资源
kind: Deployment                    # 资源类型为Deployment，用于管理Pod副本
metadata:                           # 元数据部分，包括名称、标签等
  name: eureka-app-deployment       # Deployment的名称
  labels:                           # 为该Deployment打标签
    app: eureka-app                 # 标签键为app，值为eureka-app
spec:                               # spec：定义 Deployment 的行为：如副本数、选择器、Pod 模板等
  replicas: 1                       # 指定运行1个Pod副本
  selector:                         # 定义如何选择要管理的Pod
    matchLabels:                    # 匹配具有以下标签的Pod，告诉Kubernetes，“我想要管理那些具有这些标签的所有Pod”
      app: eureka-app               # 匹配标签app=eureka-app的Pod
  template:                         # Pod模板，用于生成实际运行的Pod
    metadata:                       # Pod的元数据
      labels:                       # Pod的标签
        app: eureka-app             # 在Pod模板部分定义，用来为新创建的Pod指定标签。当你创建一个Deployment时，它会根据这个模板创建Pod，并给这些Pod打上相应的标签
    spec:                           # spec.template.spec：描述 Pod 应该如何创建（即 Pod 的规格）
      containers:                   # 容器列表，描述Pod中容器组中的每个容器应该如何运行
        - name: eureka-app          # 容器名称
          image: eureka-server:02  # 使用的镜像地址
          ports:                    # 容器暴露的端口列表
            - containerPort: 8761   # 容器监听的端口号为8761
```

> app: eureka-app 在两个地方出现是为了满足不同的目的
>
> - 第一个app: eureka-app（在selector.matchLabels中）确保了现有的、具有该标签的Pod会被控制器所管理；
> - 第二个app: eureka-app（在template.metadata.labels中）确保了根据此模板创建的新Pod也会拥有同样的标签，从而能够被同一个控制器继续管理。
> - 这两者的结合使用保证了Pod的创建和管理是一致且连贯的。如果没有这两个标签的一致性，可能会导致新的Pod不会被相应的控制器管理，或者旧的Pod不会被更新或替换。

### 2. 创建Deployment

```shell
#!/bin/bash

kubectl apply -f eureka-app-deployment.yaml

kubectl get deploy,pod
```

### 3 .查看应用的启动日志

我们可以通过kubectl logs命令来查看应用的启动日志

```shell
kubectl logs -f  pod/eureka-app-deployment-5f5d6cf5ff-gfbxv
```

![image-20250707200430806](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250707200430806.png)

## 3. **创建Service**

如果想要从外部访问应用，需要**创建Service**，添加配置文件eureka-app-service.yaml用于创建

### 1. 增加配置文件

eureka-app-service.yaml

```yml
apiVersion: v1
kind: Service
metadata:
  name: eureka-app-service
spec:
  type: NodePort
  selector:  #标签选择器，用于匹配 Pod
    app: eureka-app
  ports:
    - name: http
      protocol: TCP
      port: 8761         # Service 暴露的端口（集群内部访问）
      targetPort: 8761   # 容器实际监听的端口（Pod 中容器的端口）
```

### 2. 创建service

```shell
#!/bin/bash

kubectl apply -f eureka-app-service.yaml

kubectl get services
```



![image-20250707200748407](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250707200748407.png)

## 4. 测试访问

访问http://localhost:32272/

![image-20250707200822266](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250707200822266.png)

# 2. Kubernetes 部署需考虑的问题

我们有如下微服务：

- 消息服务：message-service

- 课程dubbo服务：course-dubbo-service

- 课程web服务：course-edge-service

- 用户thrift服务：user-thrift-service

- 用户web服务：user-edge-service

- API网关：api-gateway

## 1. Pod 的划分策略

建议的 Pod 划分方案：

- **课程服务组合**：
  - `course-dubbo-service` 和 `course-edge-service` 放在同一个 Pod 中。
  - 原因：web 服务频繁调用 dubbo 服务，放在同一 Pod 内可通过 `localhost` 直接访问，减少网络开销，提高性能。
- **消息服务**：
  - 单独部署为一个 Pod。
  - 原因：与其他服务耦合度低，独立运行便于维护和扩展。
- **用户服务组合**：
  - `user-thrift-service` 和 `user-edge-service` 放在同一个 Pod 中。
  - 原因：类似课程服务的场景，web 服务主要依赖 thrift 接口。
- **API 网关**：
  - 单独部署为一个 Pod。
  - 原因：作为所有外部请求的统一入口，需要具备高可用性和负载均衡能力。

**小结**：

> 同一业务域内、调用频繁的服务建议放在同一个 Pod 中，以提升内部通信效率；对外交互频繁或职责独立的服务应单独部署。



## 2. 同一 Pod 内服务如何互相访问

Kubernetes 中，同一个 Pod 内的所有容器共享网络命名空间，因此可以通过 `localhost` + 端口号的方式直接访问彼此的服务

```yml
# 示例：course-edge-service 调用 course-dubbo-service
dubbo:
  registry:
    address: zookeeper://localhost:2181
```

## 3. 如何让服务对外提供访问

每个对外暴露的服务都需要通过 Kubernetes 的 Service 对象来实现访问控制。

**推荐方式：**

- 使用 `ClusterIP` 类型的 Service 供内部服务之间访问；
- 使用 `NodePort` 或 `LoadBalancer` 类型的 Service 对外暴露服务；
- API 网关推荐使用 `Ingress` 控制器进行统一路由管理。

## 4. 整个系统的入口是哪个服务？如何对外暴露？

整个系统的核心入口是 **API 网关（api-gateway）**，它负责接收所有外部请求，并根据路由规则转发给对应的微服务。

**对外暴露方式：**

- 使用 `Ingress` 控制器结合域名进行统一接入；
- 如果没有 Ingress，也可以使用 `LoadBalancer` 类型的 Service；
- 可结合 TLS 加密、限流熔断等机制增强安全性。
