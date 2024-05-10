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

## 4.1 部署到服务器

构建后，部署war包到指定服务器，我这里是部署在了本地的tomcat服务上

![](https://panyuro.oss-cn-beijing.aliyuncs.com/202301152014790.png)

注意：

- war文件地址，如果不在当前项目的target下，需要写成类似 `**/target/*.war`

  ![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230115202448.png)

## 4.2 上传到私服

我这里是上传到了nexus上

![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230122205100.png)

对英需要修改setting.xml

```xml
<server>
  <id>SNAPSHOT</id>
  <username>deployment</username>
  <password>deployment123</password>
</server>

<server>
  <id>RELEASE</id>
  <username>deployment</username>
  <password>deployment123</password>
</server>
```



# 5  构建项目

如果构建失败，可以在控制台查看报错信息

![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230115202724.png)

# 6 启动tomcat

可以查看已经部署成功的服务了

![](https://panyuro.oss-cn-beijing.aliyuncs.com/202301152016375.png)

# 7. 登陆nexus

可以看到war包已经上传到了nexus

![](https://panyuro.oss-cn-beijing.aliyuncs.com/202301222054195.png)
