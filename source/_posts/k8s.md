---
title: k8s
date: 2024-01-18 16:51:58
tags:
categories:
---

# 1 K8s简介

- Kubernetes（通常简称为K8s）是一个开源的容器编排和管理平台，用于**自动化部署**、**扩展**和**管理容器化应用程序**。它提供了一组丰富的功能和工具，用于简化容器化应用程序的部署、伸缩和运维。

- K8s需要一个集群来运行和管理容器化应用程序。

- Kubernetes 集群由多个计算节点组成，其中包括主节点（Master Node）和从节点（Worker Node）。

  - 主节点负责管理和控制整个集群的状态和配置，安装了K8s的核心组件

  - 从节点是实际运行容器的计算节点。
    - 从节点（Worker Node）上的容器化环境一般是通过 Docker 运行的。
    - 每个 Worker Node 上可以运行多个容器， 其中每个容器都可以托管一个或多个 Pod。

<img src="https://panyuro.oss-cn-beijing.aliyuncs.com/image-20240122164513407.png" alt="image-20240122164513407" style="zoom:70%;" />

# 2 重要概念

## 2.1 Pod

- Pod 是一种逻辑概念，用于组织和管理容器的基本单位。
- 它是一个抽象的概念，用于封装一个或多个相关的容器、存储卷、网络和其他资源
- 所有的应用，服务最终都是运行在pod上，pod是一个容器
- pod有一个独立的ip，pod里的容器可以共享网络，共享IP
- pod里面可以有任意多个容器，以及任意多个存储(volumn)

<img src="https://panyuro.oss-cn-beijing.aliyuncs.com/image-20240122142025469.png" alt="image-20240122142025469" style="zoom:50%;" />

- k8s调度pod, 把pod运行起来
  - Pod在node上运行，一个Node上可以有任意多个pod

<img src="https://panyuro.oss-cn-beijing.aliyuncs.com/image-20240122143320022.png" alt="image-20240122143320022" style="zoom:33%;" />

## 2.2 service

- service是一个逻辑层的概念，将一堆pod通过打标签的方式，划分成逻辑组，从而方便实现负载均衡

- 下图三个pod对应一个service B，通常表示一个应用的多个副本，即对同一个应用扩容，从一个实例变成了三个，他们对外提供相同的服务。通过service的IP可以对多个Pod的地址进行负载均衡

  > 灰度发布的时候，可以先将一个版本改掉，再依次改其他的

<img src="https://panyuro.oss-cn-beijing.aliyuncs.com/image-20240122144759597.png" alt="image-20240122144759597" style="zoom:50%;" />

- 通过打标签的方式识别Pod属于哪个service

<img src="https://panyuro.oss-cn-beijing.aliyuncs.com/image-20240122145618560.png" alt="image-20240122145618560" style="zoom:50%;" />



> 为什么Pod已经有了IP还需要service再包一层呢？
>
> - 当Pod出现意外挂掉时，这个服务无法访问，他肯定会去其他位置新启动一个服务，通过service的IP依旧可以找到这个新建的服务

## 2.3 Deployment

- Deployment 可以部署service，也可以部署Pod

  这张图可以理解为：在master上发起了一个deployment，选中了三个节点中的一个，然后部署了一个新的应用，运行在容器中

<img src="https://panyuro.oss-cn-beijing.aliyuncs.com/image-20240122141719723.png" alt="image-20240122141719723" style="zoom: 50%;" />

- 怎么通过deployment完成应用扩容？
  1. 从master发起一个，想给service里的一个Pod，扩容成4个实例
  2. 在其他节点上启动pod，通过打标签将这组pod标识为同一个service
  3. service可以感知哪个pod出现问题，然后就不会把流量转发给他
- 滚动更新的过程：停掉一个pod启动一个，停掉一个启动一个

# 3 基础集群部署

## 1. 部署ETCD（主节点）

 kubernetes需要存储很多东西，像它本身的节点信息，组件信息，还有通过kubernetes运行的pod，deployment，service等等。都需要持久化。etcd就是它的数据中心。生产环境中为了保证数据中心的高可用和数据的一致性，一般会部署最少三个节点。

端口：2379， 2380

## 2. 部署APIServer（主节点）

kube-apiserver是Kubernetes最重要的核心组件之一，主要提供以下的功能

- 提供集群管理的REST API接口，包括认证授权（我们现在没有用到）数据校验以及集群状态变更等
- 提供其他模块之间的数据交互和通信的枢纽（其他模块通过API Server查询或修改数据，只有API Server才直接操作etcd）

> 生产环境为了保证apiserver的高可用一般会部署2+个节点，在上层做一个lb做负载均衡，比如haproxy。

端口：6443， 8080

## 3. 部署ControllerManager（主节点）

- Controller Manager是Kubernetes的大脑，它通过apiserver监控整个集群的状态，并确保集群处于预期的工作状态。 

- Controller Manager由kube-controller-manager和cloud-controller-manager组成。
  - kube-controller-manager由一系列的控制器组成，像Replication Controller控制副本，Node Controller节点控制，Deployment Controller管理deployment等等
  - cloud-controller-manager在Kubernetes启用Cloud Provider的时候才需要，用来配合云服务提供商的控制

## 4. 部署Scheduler（主节点）

kube-scheduler负责分配调度Pod到集群内的节点上，它监听kube-apiserver，查询还未分配Node的Pod，然后根据调度策略为这些Pod分配节点。kubernetes的各种调度策略就是它实现的。

## 5. 部署CalicoNode（所有节点）

Calico实现了CNI接口，是kubernetes网络方案的一种选择，它一个纯三层的数据中心网络方案（不需要Overlay），并且与OpenStack、Kubernetes、AWS、GCE等IaaS和容器平台都有良好的集成。 

Calico在每一个计算节点利用Linux Kernel实现了一个高效的vRouter来负责数据转发，而每个vRouter通过BGP协议负责把自己上运行的workload的路由信息像整个Calico网络内传播——小规模部署可以直接互联，大规模下可通过指定的BGP route reflector来完成。 这样保证最终所有的workload之间的数据流量都是通过IP路由的方式完成互联的。

## 6. 配置kubectl命令（任意节点）

kubectl是Kubernetes的命令行工具，是Kubernetes用户和管理员必备的管理工具。 kubectl提供了大量的子命令，方便管理Kubernetes集群中的各种功能。

## 7. 配置kubelet（工作节点）

每个工作节点上都运行一个kubelet服务进程，默认监听10250端口，接收并执行master发来的指令，管理Pod及Pod中的容器。每个kubelet进程会在API Server上注册节点自身信息，定期向master节点汇报节点的资源使用情况，并通过cAdvisor监控节点和容器的资源。

# 四、kubernetes集群部署微服务

##### 我们有如下微服务：

- 消息服务：message-service
- 课程dubbo服务：course-dubbo-service
- 课程web服务：course-edge-service
- 用户thrift服务：user-thrift-service
- 用户web服务：user-edge-service
- API网关：api-gateway

##### 把它们放到kubernetes集群运行我们要考虑什么问题？

- 哪些服务适合单独成为一个pod？哪些服务适合在一个pod中？

  如果把多个服务放在一个pod里，他们之间的访问效率会高。直接可以通过localhost访问在同一pod的其他服务

  - 课程web服务的接口主要是课程dubbo服务调用，故他们两个放在一个pod
  - 消息服务以及API网关和其他服务关系不大，所以彼此单独放在一个pod

- 在一个pod里面的服务如何彼此访问？他们的服务如何对外提供服务？

- 单独的pod如何对外提供服务？

- 哪个服务作为整个服务的入口，入口服务如何对外提供服务？



每个微服务下都有一个Dockerfile以及一个build.sh ,build.sh 就是执行docker build ,并且将打包好的镜像放到docker仓库上
