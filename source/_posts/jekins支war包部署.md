---
title: jenkins之war包部署
date: 2023-01-15 19:46:19
tags:
categories: jenkins
---

# 1. 插件安装

- Maven Integration

- Pipeline Maven Integration

- Deploy to container

![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230115200509.png)

![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230115200537.png)



# 2. 新建任务

![](https://panyuro.oss-cn-beijing.aliyuncs.com/202301152009848.png)

# 3 源码配置

- Credentials 需要选择在用户列表，配置了该仓库ssh key的用户

![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230115201137.png)

# 4 构建后操作

构建后，部署war包到指定服务器，我这里是部署在了本地的tomcat服务上

![](https://panyuro.oss-cn-beijing.aliyuncs.com/202301152014790.png)

注意：

- war文件地址，如果不在当前项目的target下，需要写成类似 `**/target/*.war`

  ![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230115202448.png)

# 5  构建项目

如果构建失败，可以在控制台查看报错信息

![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230115202724.png)

# 6 启动tomcat

可以查看已经部署成功的服务了

![](https://panyuro.oss-cn-beijing.aliyuncs.com/202301152016375.png)
