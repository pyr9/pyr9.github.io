---
title: nexus私服搭建与核心功能
date: 2023-01-15 20:42:13
tags:
categories: maven
---

# 1. 私服使用场景

分布式系统，多个项目依赖的共享

# 2 nexus安装

1. 进入[Repository Manager 2 (sonatype.com)](https://help.sonatype.com/repomanager2)，选择对应的版本进行下载

2. 下载解压后，进入bin目录，运行 `./nexus start`

3. 访问http://localhost:8081/nexus

   ![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230116212514.png)

注：nexus启动失败，查看报错日志（nexus-2.15.1-02/logs/wrapper.log）

![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230116212959.png)

修复：将javax.activation-1.2.0.jar 和jaxb-api-2.3.0.jar包放到nexus的lib下

4  登陆nexus，nexus2 默认用户名和密码是`admin` and `admin123` .不确定的可以查看官方文档[Running (sonatype.com)](https://help.sonatype.com/repomanager2/installing-and-running/running)

![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230116215507.png)

5. 查看仓库

   ![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230116220006.png)
