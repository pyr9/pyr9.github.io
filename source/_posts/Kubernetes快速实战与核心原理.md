---
title: Kubernetes核心原理
date: 2025-06-29 16:51:58
tags:
Categories: Kubernetes
---

# 1.  K8s简介

Kubernetes（通常简称为K8s）是一个开源的容器编排和管理平台，用于**自动化部署**、**扩展**和**管理容器化应用程序**。它提供了一组丰富的功能和工具，用于简化容器化应用程序的部署、伸缩和运维。翻译成大白话就是：**K8S 是负责自动化运维管理多个Docker 程序的集群**。

# 2. **K8s核心特性**

- 服务发现与负载均衡：无需修改你的应用程序即可使用陌生的服务发现机制。
- 存储编排：自动挂载所选存储系统，包括本地存储。
- Secret和配置管理：部署更新Secrets和应用程序的配置时不必重新构建容器镜像，且不必将软件堆栈配置中的秘密信息暴露出来。

- 批量执行：除了服务之外，Kubernetes还可以管理你的批处理和CI工作负载，在期望时替换掉失效的容器。
- 水平扩缩：使用一个简单的命令、一个UI或基于CPU使用情况自动对应用程序进行扩缩。
- 自动化上线和回滚：Kubernetes会分步骤地将针对应用或其配置的更改上线，同时监视应用程序运行状况以确保你不会同时终止所有实例。
- 自动装箱：根据资源需求和其他约束自动放置容器，同时避免影响可用性。
- 自我修复：重新启动失败的容器，在节点死亡时替换并重新调度容器，杀死不响应用户定义的健康检查的容器。

# 3. K8s组成

K8s需要一个集群来运行和管理容器化应用程序。Kubernetes 集群由多个计算节点组成，其中包括主节点（Master Node）和从节点（Worker Node）。

- 主节点负责管理和控制整个集群的状态和配置，安装了K8s的核心组件

- 从节点是实际运行容器的计算节点。
  - 从节点（Worker Node）上的容器化环境一般是通过 Docker 运行的。
  - 每个 Worker Node 上可以运行多个容器， 其中每个容器都可以托管一个或多个 Pod。

<img src="https://panyuro.oss-cn-beijing.aliyuncs.com/image-20240122164513407.png" alt="image-20240122164513407" style="zoom:70%;" />

# 4. K8s核心概念

## 4.1 Deployment

Deployment负责创建和更新应用程序的实例。创建Deployment后，Kubernetes Master 将应用程序实例调度到集群中的各个节点上。如果托管实例的节点关闭或被删除，Deployment控制器会将该实例替换为群集中另一个节点上的实例。这提供了一种自我修复机制来解决机器故障维护问题。

- Deployment 可以部署service，也可以部署Pod。这张图可以理解为：在master上发起了一个deployment，选中了三个节点中的一个，然后部署了一个新的应用，运行在容器中


<img src="https://panyuro.oss-cn-beijing.aliyuncs.com/image-20240122141719723.png" alt="image-20240122141719723" style="zoom: 50%;" />

- 怎么通过deployment完成应用扩容？
  1. 从master发起一个，想给service里的一个Pod，扩容成4个实例
  2. 在其他节点上启动pod，通过打标签将这组pod标识为同一个service
  3. service可以感知哪个pod出现问题，然后就不会把流量转发给他
- 滚动更新的过程：停掉一个pod启动一个，停掉一个启动一个

## 4.2 Pod

Pod相当于**逻辑主机**的概念，负责托管应用实例。包括一个或多个应用程序容器（如 Docker），以及这些容器的一些共享资源（共享存储、网络、运行信息等）。

- Pod 是一种逻辑概念，用于组织和管理容器的基本单位, k8S 对docker进行了一些封装, 就成了pod。它是一个抽象的概念，用于封装一个或多个相关的容器、存储卷、网络和其他资源
- 所有的应用，服务最终都是运行在pod上，pod是一个容器
- pod有一个独立的ip，pod里的容器可以共享网络，共享IP
- pod里面可以有任意多个容器，以及任意多个存储(volumn)

<img src="https://panyuro.oss-cn-beijing.aliyuncs.com/image-20240122142025469.png" alt="image-20240122142025469" style="zoom:50%;" />

- k8s调度pod, 把pod运行起来。Pod在node上运行，一个Node上可以有任意多个pod。

<img src="https://panyuro.oss-cn-beijing.aliyuncs.com/image-20240122143320022.png" alt="image-20240122143320022" style="zoom:33%;" />



## 4.3 service

service是一个逻辑层的概念，将一堆pod通过打标签的方式，划分成逻辑组，从而方便实现负载均衡. 尽管每个Pod 都有一个唯一的IP地址，但是如果没有Service，这些IP不会暴露在群集外部。Service允许您的应用程序接收流量。

- Service也可以用在ServiceSpec标记type的方式暴露，type类型如下：

  - ClusterIP（默认）：在集群的内部IP上公开Service。这种类型使得Service只能从集群内访问。
  - NodePort：使用NAT在集群中每个选定Node的相同端口上公开Service。使用 <NodeIP>:<NodePort> 从集群外部访问Service。是ClusterIP的超集。
  - LoadBalancer：在当前云中创建一个外部负载均衡器(如果支持的话)，并为Service分配一个固定的外部IP。是NodePort的超集。

  - ExternalName：通过返回带有该名称的CNAME记录，使用任意名称（由spec中的externalName指定）公开Service。不使用代理。

- 下图三个pod对应一个service B，通常表示一个应用的多个副本，即对同一个应用扩容，从一个实例变成了三个，他们对外提供相同的服务。通过service的IP可以对多个Pod的地址进行负载均衡

  > 灰度发布的时候，可以先将一个版本改掉，再依次改其他的

<img src="https://panyuro.oss-cn-beijing.aliyuncs.com/image-20240122144759597.png" alt="image-20240122144759597" style="zoom:50%;" />

- 通过打标签的方式识别Pod属于哪个service

<img src="https://panyuro.oss-cn-beijing.aliyuncs.com/image-20240122145618560.png" alt="image-20240122145618560" style="zoom:50%;" />



> 为什么Pod已经有了IP还需要service再包一层呢？
>
> - 当Pod出现意外挂掉时，这个服务无法访问，他肯定会去其他位置新启动一个服务，通过service的IP依旧可以找到这个新建的服务

# 5. **K8s 核心架构原理**

K8S 是属于**主从设备模型（Master-Slave 架构）**，即有 Master 节点负责核心的调度、管理和运维，Slave 节点则执行用户的程序。但是在 K8S 中，主节点一般被称为**Master Node 或者 Head Node**，而从节点则被称为**Worker Node 或者 Node**。

![image-20250701214938688](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250701214938688.png)

注意：Master Node 和 Worker Node 是分别安装了 K8S 的 Master 和 Woker 组件的实体服务器，每个 Node都对应了一台实体服务器（虽然 Master Node 可以和其中一个 Worker Node 安装在同一台服务器，但是建议Master Node 单独部署），**所有 Master Node 和 Worker Node 组成了 K8S 集群**，同一个集群可能存在多个Master Node 和 Worker Node

## 5.1  **Master Node**的组件

- **API Server**。**K8S 的请求入口服务**。API Server 负责接收 K8S 所有请求（来自 UI 界面或者 CLI 命令行工具），然后，API Server 根据用户的具体请求，去通知其他组件干活。

- **Controller Manager**。**K8S 所有 Worker Node 的监控器**。Controller Manager 有很多具体的Controller， Node Controller、Service Controller、Volume Controller 等。Controller 负责监控和调整在 Worker Node 上部署的服务的状态，比如用户要求 A 服务部署 2 个副本，那么当其中一个服务挂了的时候，Controller 会马上调整，让 Scheduler 再选择一个 Worker Node 重新部署服务。

- **etcd**。**K8S 的存储服务**。etcd 存储了 K8S 的关键配置和用户配置，K8S 中仅 API Server 才具备读写权限，其他组件必须通过 API Server 的接口才能读写数据。
- **Scheduler**。**K8S 所有 Worker Node 的调度器**。当用户要部署服务时，Scheduler 会选择最合适的Worker Node（服务器）来部署。

## 5.2  **Worker Node**的组件

- **Kubelet**。**Worker Node 的监视器，以及与 Master Node 的通讯器**。Kubelet 是 Master Node 安插在 Worker Node 上的“眼线”，它会定期向 Master Node 汇报自己 Node 上运行的服务的状态，并接受来自 Master Node 的指示采取调整措施。负责控制所有容器的启动停止，保证节点工作正常。
- **Kube-Proxy**。**K8S 的网络代理**。Kube-Proxy 负责 Node 在 K8S 的网络通讯、以及对外部网络流量的负载均衡。
- **Container Runtime**。**Worker Node 的运行环境**。即安装了容器化所需的软件环境确保容器化程序能够跑起来，比如 Docker Engine运行环境。

## 5.3 各组件协同工作的过程

以用K8S部署Nginx的过程为例, **我们在master节点执行一条命令要master部署一个nginx应用**

```shell
kubectl create deployment nginx --image=nginx
```

1. 这条命令首先发到master节点的网关api server，这是matser的唯一入口

2. api server将命令请求交给controller mannager进行控制

3. controller mannager 进行应用部署解析.  controller mannager 会生成一次部署信息，并通过api server将信息存入etcd存储中

4. scheduler调度器通过api server从etcd存储中，拿到要部署的应用，开始调度看哪个节点有资源适合部署

5. scheduler把计算出来的调度信息通过api server再放到etcd中

6. 每一个node节点的监控组件kubelet，随时和master保持联系（给api-server发送请求不断获取最新数据），拿到master节点存储在etcd中的部署信息

7. 假设node2的kubelet拿到部署信息，显示他自己节点要部署某某应用. kubelet就自己run一个应用在当前机器上，并随时给master汇报当前应用的状态信息,  node和master也是通过master的api-server组件联系的

# 6. K8s基础集群部署

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

# 7. k8s集群部署微服务

- 我们有如下微服务：

  - 消息服务：message-service

  - 课程dubbo服务：course-dubbo-service

  - 课程web服务：course-edge-service

  - 用户thrift服务：user-thrift-service

  - 用户web服务：user-edge-service

  - API网关：api-gateway


- 把它们放到kubernetes集群运行我们要考虑什么问题？

  - 哪些服务适合单独成为一个pod？哪些服务适合在一个pod中？

    如果把多个服务放在一个pod里，他们之间的访问效率会高。直接可以通过localhost访问在同一pod的其他服务

    - 课程web服务的接口主要是课程dubbo服务调用，故他们两个放在一个pod
    - 消息服务以及API网关和其他服务关系不大，所以彼此单独放在一个pod


  - 在一个pod里面的服务如何彼此访问？他们的服务如何对外提供服务？


  - 单独的pod如何对外提供服务？


  - 哪个服务作为整个服务的入口，入口服务如何对外提供服务？


每个微服务下都有一个Dockerfile以及一个build.sh ,build.sh 就是执行docker build ,并且将打包好的镜像放到docker仓库上
