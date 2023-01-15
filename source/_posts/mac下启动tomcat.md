---
title: mac下启动tomcat
date: 2023-01-15 20:45:56
tags:
categories: tomcat
---

# 1. 安装

登录Apache Tomcat官网，地址 [http://tomcat.apache.org](http://tomcat.apache.org/) ，左边的Download，点击选择需要下载的版本 Tomcat8

![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230115204748.png)

## 2 启动

下载下来解压，进入bin目录，运行`./startup.sh`

![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230115205133.png)

停止运行 `./shutdown.sh`

# 3 修改默认端口号

不修改的话，可以直接访问8080端口，我这里修改成了8888

1.  进入conf 目录 `cd conf`
2. 修改server.xml  `vim server.xml`

![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230115205500.png)

# 4 设置用户信息 

如果需要manager App，还需要增加用户名和密码

1.  进入conf 目录 `cd conf`

2. 修改server.xml  `vim tomcat-users.xml`

   ![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230115205918.png)

# 5 访问 8888端口

![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230115210632.png)
