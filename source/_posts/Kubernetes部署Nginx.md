---
title: Kubernetes部署Nginx
date: 2025-07-01 22:00:09
tags:
Categories: Kubernetes
---

# 1. **K8S部署Nginx**

在k8s-master机器上执行

## 1. 创建一次deployment部署

```shell
kubectl create deployment nginx ‐‐image=nginx
```

<img src="https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250703222411043.png" alt="image-20250703222411043" style="zoom:100%;" />

## 2. 创建 Service

使用 `kubectl expose` 来为之前创建的名为 `nginx` 的 Deployment 创建一个 Service

```shell
kubectl expose deployment nginx ‐‐port=80 ‐‐type=NodePort
```

![image-20250703222504494](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250703222504494.png)

## 3. 访问Nginx地址

 http://localhost:30918/

![image-20250703221802668](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250703221802668.png)

## 4. **对nginx这个deployment进行扩缩容**

```shell
kubectl scale --replicas=2 deployment nginx
```

![image-20250703224359533](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250703224359533.png)

## 5. 滚动升级

```shell
kubectl set image deployment.apps/nginx  nginx=nginx:1.28.0
```

- `deployment.apps/nginx`: 指定了要操作的目标资源类型（Deployment）及其名称（nginx）。
- `nginx=nginx:1.28.0`: 这部分指定了容器名和新的镜像名称及标签。其中 `nginx` 是 Deployment `nginx` 中容器的名字，而 `nginx:1.28.0` 是你希望更新到的新镜像。

使用 `curl` 发送请求：查看响应头中的 `Server` 字段来获取 Nginx 版本信息

![image-20250705115801052](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250705115801052.png)

查看某个pod的详细信息，发现pod里的镜像版本已经升级了

![image-20250703230511156](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250703230511156.png)

## 6. 版本回滚

### 1. 查看历史版本

```shell
kubectl rollout history deploy nginx
```

![image-20250705120109209](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250705120109209.png)

### 2. 回滚到上一个版本

```shell
 kubectl rollout undo deploy nginx   #‐‐to‐revision 参数可以指定回退的版本
```

![image-20250705120156769](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250705120156769.png)

再次使用 `curl` 访问nginx，发现版本已经回退

![image-20250705120352860](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250705120352860.png)

## 7, **标签的使用**

通过给资源添加Label，可以方便地管理资源（如Deployment、Pod、Service等）。

### 1. 查看Deployment中所包含的Label

![image-20250705120548151](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250705120548151.png)

### 2. 通过Label查询Pod

```
kubectl get pods -l app=nginx
```

![image-20250705155913531](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250705155913531.png)

### 3. 通过Label查询Service

```
kubectl get services ‐l app=nginx
```

![image-20250705155936913](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250705155936913.png)

### 4. 给Pod添加Label

```shell
kubectl label pod my‐tomcat‐685b8fd9c9‐lrwst version=test-label
```

### 5. 通过Label查询Pod

```shell
 kubectl get pods ‐l version=test-label
```

![image-20250705160210458](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20250705160210458.png)

### 6. 通过Label删除服务

```
kubectl delete service ‐l app=test-label
```

