---
title: Eureka注册服务IP的方式
date: 2025-06-03 22:02:49
tags:
categories:
---

# 1. Eureka注册服务IP的方式

当服务注册到Eureka、Nacos等注册中心时，它会将自己的IP地址上报。Spring Boot默认可能会使用如下几种方式来获取本机 IP:

- localhost

- 网卡上的IP(如:192.168.x.x ，172.17.x.x , 10.x.x.x)

- 如果设置了preferred-networks，则优先从这些网段中选一个作为注册地址

  如果没有设置这个选项，有时会错误地将localhost或者错误的IP注册到服务中心，导致其他服务无法访问。

# 2. 解决注册到Eureka一直显示localhost的问题

```java
-Dspring.cloud.inetutils.preferred-networks=172.28
```

