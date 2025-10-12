---
title: k3s部署Java项目
date: 2025-9-12 19:12:55
tags:
categories: Kubernetes
---

# 1. 环境准备

当前Java项目是使用docker安装的，想切换成k3s安装，两台Linux机器，搭建主从

| 节点  | IP           | 角色   |
| ----- | ------------ | ------ |
| node1 | 172.28.49.11 | master |
| node2 | 172.28.49.13 | worker |

# 2. K3s安装

## 2.1.  安装 k3s 主节点（Master）  

### 2.1.1. 执行安装命令

```plain
curl -sfL https://get.k3s.io | \
  K3S_KUBECONFIG_MODE="644" \
  sh -s - server \
  --node-name test-app-for-mes 
```

- 使用--node-name test-app-for-mes 是为了设置主机名，因为主机名实际是test_app_for_mes, **违反了 Kubernetes 的命名规范（如未违反，可不用设置）**

### 2.1.2. 查看当前状态

需要是active， 且日志不报错

```plain
systemctl status k3s
kubectl get nodes -A
```

![img](https://panyuro.oss-cn-beijing.aliyuncs.com/1759213852952-f55e33fb-9dc9-437f-8e33-29cec570acd3.png)



## 2.2.  安装 k3s worker（从节点）  

### 2.2.1. 主节点获取Join token

```plain
cat /var/lib/rancher/k3s/server/node-token
```

### 2.2.2. 安装agent

```shell
curl -sfL https://get.k3s.io | \
  K3S_URL=https://172.28.49.11:6443 \
  K3S_TOKEN=K10032798e8d89a392dc81fa66822edc9d3bb8d660be060bdef71df5a2d0269a396::server:bd8ef4a1a6a0c514df13a24e39b445df \
  INSTALL_K3S_EXEC="--node-name test-midd-for-mes" \
  sh -
```

### 2.2.3. 验证集群状态

```plain
kubectl get nodes
```

![img](https://panyuro.oss-cn-beijing.aliyuncs.com/1759213804109-204ac34a-4824-492c-beab-3be5e0d485f1.png)



PS：如果执行卡住，可以在新开一个窗口，查看日志原因：journalctl -u k3s-agent -f

# 3.  编写 Kubernetes 资源  

## 3.1.  Deployment  

mycim-webapp-deployment.yaml

```plain
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mycim-webapp-deployment
  namespace: mycim
  labels:
    app: mycim-webapp-dev
spec:
  replicas: 4
  selector:
    matchLabels:
      app: mycim-webapp-dev
  template:
    metadata:
      labels:
        app: mycim-webapp-dev
    spec:
      containers:
      - name: mycim-webapp-dev
        image: docker.xxx.com/ja-zjmes/dev/webapp/qj-mycim:10d2be77
        ports:
        - containerPort: 8080
        env:
          - name: TZ
            value: "Asia/Shanghai"
        volumeMounts:
        - name: localtime-volume
          mountPath: /etc/localtime
          readOnly: true
        - name: config-volume
          mountPath: /home/mycim/config
        - name: logs-volume
          mountPath: /home/mycim/logs
        livenessProbe:
          tcpSocket:
            port: 8080
          initialDelaySeconds: 120
          periodSeconds: 10
        readinessProbe:
          tcpSocket:
            port: 8080
          initialDelaySeconds: 120
          periodSeconds: 5
      volumes:
        - name: localtime-volume
          hostPath:
            path: /etc/localtime
        - name: config-volume
          hostPath:
            path: /home/xxx/qj-mycim/config
        - name: logs-volume
          hostPath:
            path: /home/xxx/qj-mycim/logs
```

## 3.2. Service

mycim-webapp-service.yaml

```plain
#Service 服务发现
apiVersion: v1
kind: Service
metadata:
  name: mycim-webapp-dev-service
  namespace: mycim
spec:
  selector:
    app: mycim-webapp-dev
  ports:
    - protocol: TCP
      port: 8080    # ClusterIP 端口
      targetPort: 8080     # Pod 上的实际端口
```

## 3.3.  Ingress （可选）

如果你想通过域名/宿主机 80 访问：

 mycim-webapp-ingress.yaml

```plain
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mycim-webapp-dev-ingress
  namespace: mycim
  annotations:
    kubernetes.io/ingress.class: "traefik"
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: mycim-webapp-dev-service
            port:
              number: 8080
```

使用到了 k3s 内置的Traefik , 所以需要保证它正常运行，可执行

kubectl get pods -n kube-system | grep traefik

# 4.  部署到 k3s  

```plain
kubectl create namespace mycim
kubectl apply -f mycim-webapp-deployment.yaml
kubectl apply -f mycim-webapp-service.yaml
kubectl apply -f mycim-webapp-ingress.yaml
```

- 查看 Pod 状态：

```plain
kubectl get pods -o wide
kubectl get svc
kubectl get ingress
```

![img](https://panyuro.oss-cn-beijing.aliyuncs.com/1759217367871-8479a8b9-9779-40e5-a92b-5603e66ebf27.png)

![img](https://panyuro.oss-cn-beijing.aliyuncs.com/1759217399413-2fc24366-25ea-471a-a7ee-415e47e4e0c4.png)

# 5. 常见问题

## 5.1.  核心镜像拉取失败

### 5.1.1. 详细报错

Warning  FailedCreatePodSandBox  11m (x316 over 3h36m)  kubelet  (combined from similar events): Failed to create pod sandbox: rpc error: code = DeadlineExceeded desc = failed to start sandbox "90b5b09ba2e907a2fc5a117916e8b421b74556892c7ae8764b77f729c4c31934": failed to get sandbox image "rancher/mirrored-pause:3.6": failed to pull image "rancher/mirrored-pause:3.6": failed to pull and unpack image "docker.io/rancher/mirrored-pause:3.6": failed to resolve reference "docker.io/rancher/mirrored-pause:3.6": failed to do request: Head "https://registry-1.docker.io/v2/rancher/mirrored-pause/manifests/3.6": dial tcp 128.242.240.212:443: i/o timeout

### 5.1.2. 解决方式：

1. 修改镜像源

```shell
sudo mkdir -p /etc/rancher/k3s
sudo touch /etc/rancher/k3s/registries.yaml
vim /etc/rancher/k3s/registries.yaml
mirrors:
  docker.io:
    endpoint:
      - https://docker.1ms.run
      - https://docker.xuanyuan.me
  rancher:
    endpoint:
      - https://docker.1ms.run
      - https://docker.xuanyuan.me
```

1. 重启k3s

```shell
systemctl restart k3s
```

注：如果不生效，可尝试删除/var/lib/rancher/k3s/（谨慎！！）， 避免缓存影响

## 5.2. 不想使用80端口访问

traefik 默认暴露的是80和443，由于网络策略原因需要修改成宿主机通过http://172.28.49.11:8080/ 访问。

1. 修改端口

```plain
kubectl -n kube-system edit svc traefik
```

ports:

  \- name: web

​    nodePort: 31869

​    port: 8080 (本来是80)

​    protocol: TCP

​    targetPort: web



1. 查看是否修改成功

```plain
kubectl -n kube-system get svc traefik
```

![img](https://panyuro.oss-cn-beijing.aliyuncs.com/1759214761756-8a2d0850-e891-418a-bb12-3769e2b62d2a.png)



# 6. 常用命令

## 6.1. 集群管理

- **停止 K3s 服务**：**bash**

```bash
systemctl stop k3s
```

- **重启 K3s 服务**：**bash**

```bash
systemctl restart k3s
```

- **查看 K3s 服务状态**：**bash**

```bash
systemctl status k3s
```

## 6.2. 应用部署与管理

- **部署应用**（通过 YAML 文件）：**bash**

```bash
kubectl apply -f deployment.yaml
```

- **查看部署状态**：**bash**

```bash
kubectl get deployments
```

- **查看 Pod 状态**：**bash**

```bash
kubectl get pods
# 查看所有命名空间的 Pod
kubectl get pods --all-namespaces
```

- **查看 Pod 详细信息**：**bash**

```bash
kubectl describe pod <pod-name>
```

- **查看服务（Service）**：**bash**

```bash
kubectl get services
```

- **删除部署**：**bash**

```bash
kubectl delete deployment <deployment-name>
```

- 修改默认namespace

```shell
kubectl config set-context --current --namespace=mycim
```

## 6.3. 日志与调试

- **查看 Pod 日志**：**bash**

```bash
# 实时查看日志
kubectl logs <pod-name> -f
```

- **查看 K3s 服务器日志**：**bash**

```bash
journalctl -u k3s -f
```

- **查看工作节点日志**：**bash**

```bash
journalctl -u k3s-agent -f
```

## 6.4. 配置与信息查询

- **查看集群信息**：**bash**

```bash
kubectl cluster-info
```

- **查看 K3s 配置**：**bash**

```bash
sudo cat /etc/rancher/k3s/k3s.yaml
```

- **获取命名空间**：**bash**

```bash
kubectl get namespaces
```

## 6.5. 其他常用操作

- **查看资源使用情况**：**bash**

```bash
kubectl top nodes
kubectl top pods
```

- **执行 Pod 内命令**：**bash**

```bash
kubectl exec -it <pod-name> -- /bin/sh
```

# 
