---
title: docker部署java服务踩坑记录
date: 2025-06-18 21:47:53
tags:
categories: docker
---

# 1. java.net.UnknownHostException

## 1.1. 问题

```plain
InetAddress address = InetAddress.getLocalHost();
```

在上述代码中，我们尝试通过 `InetAddress.getHostAddress()` 方法获取主机名对应的Host地址。如果主机名无法解析，将抛出 `UnknownHostException` 异常。

**错误日志日志截图**

![img](https://panyuro.oss-cn-beijing.aliyuncs.com/1750210808068-80321ab6-8263-419d-a28d-42d8abae6cc6.png)



## 1.2. 解决

修改hosts文件

```plain
vim /etc/hosts
10.108.200.83 ZJ-Test-Mycim
```

# 2. 时区提前了8小时

## 2.1. 问题

- 日志时间提前了8小时
- 业务逻辑中使用了时间比较的会报错，时间还没到

```plain
String currentTime = formatTime.format(new Date());
```

## 2.2. 解决

增加时区映射

```
  -e TZ=Asia/Shanghai \
  -v /etc/localtime:/etc/localtime:ro \
```

# 3. No such file or directory

## 3.1. 问题

文件创建失败，报错/home/oracle/ftp/e2feea11-7d6e-4e51-95d0-b51fc6f16d27.txt (No such file or directory)"

```java
File fileOut = new File("/home/oracle/ftp/"+txtUuid+".txt");
```

## 3.2. 解决

容器里创建了/home/oracle/ftp/目录，并且映射到了宿主机

# 4. 与其他系统通信失败

## 4.1. 问题

服务调用失败， 不识别bridge下的服务IP

## 4.2. 解决

使用host模式启动服务

# 5. docker in docker (jenkins)

## 5.1. 问题

在docker启动的Jenkins里无法识别docker命令

## 5.2 解决

增加映射

```
- /var/run/docker.sock:/var/run/docker.sock
- /usr/bin/docker:/usr/bin/docker
- /etc/docker:/etc/docker
```



