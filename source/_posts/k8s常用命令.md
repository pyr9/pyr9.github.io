---
title: k8s常用命令
date: 2024-02-04 19:25:32
tags:
categories:
---

- #### 1. 用频率最高的K8s常用命令

  - `kubectl get`: 获取资源的信息，如获取Pod、Service、Deployment等资源的状态信息。
  - `kubectl create`: 创建资源，如创建Pod、Service、Deployment等资源。
  - `kubectl delete`: 删除资源，如删除Pod、Service、Deployment等资源。
  - `kubectl apply`: 应用配置文件，如应用Deployment的配置文件。
  - `kubectl describe`: 查看资源的详细信息，如查看Pod、Service、Deployment等资源的详细配置和状态信息。

  #### 2. 难度较高的K8s常用命令

  - `kubectl exec`: 在容器内部执行命令，如在Pod内部执行命令或访问容器内部的终端。
  - `kubectl port-forward`: 将集群内的服务端口转发到本地，用于本地访问集群内的服务。
  - `kubectl logs`: 查看Pod的日志信息，如查看容器的标准输出和标准错误输出。
  - `kubectl scale`: 调整资源的副本数，如调整Deployment的副本数。
  - `kubectl rollout`: 控制应用的滚动更新，如进行版本升级或回滚。

  #### 3. 易错的K8s常用命令

  - `kubectl get pods`: 获取Pod的信息时，常常忘记加`s`，导致无法获取到Pod的状态信息。
  - `kubectl create -f <file>`: 创建资源时，忘记指定配置文件，导致资源无法创建成功。
  - `kubectl delete pod <pod-name>`: 删除Pod时，忘记指定Pod的名称，导致无法删除指定的Pod。
  - `kubectl apply -f <file>`: 应用配置文件时，忘记指定配置文件，导致配置文件无法生效。
  - `kubectl describe <resource>`: 查看资源的详细信息时，忘记指定资源的名称，导致无法获取到详细信息。

  #### 4. 其他命令

  ```
  Kubernetes（K8s） 常用命令~
  ```

  1. kubectl get pods：获取当前集群中所有的Pods。
  2. kubectl describe pod [pod名称]：显示指定Pod的详细信息。
  3. kubectl create -f [yaml文件]：使用yaml文件创建一个资源（如Pod、Deployment等）。
  4. kubectl apply -f [yaml文件]：使用yaml文件创建或更新一个资源。
  5. `kubectl delete pod [pod名称]`：删除指定的Pod。
  6. kubectl scale deployment [deployment名称] --replicas=[副本数量]：扩展或缩减指定Deployment的副本数量。
  7. kubectl exec -it [pod名称] [命令]：在指定的Pod中执行命令。
  8. kubectl logs [pod名称]：查看指定Pod的日志。
  9. kubectl port-forward [pod名称] [本地端口]:[远程端口]：将本地端口与Pod中的端口进行转发。
  10. kubectl get deployments：获取当前集群中所有的Deployments。
