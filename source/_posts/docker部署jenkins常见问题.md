---
title: docker部署jenkins常见问题
date: 2023-10-03 18:41:50
tags:
categories: jenkins
---

# 1 需要迁移的文件夹

将老服务器[jenkins](https://so.csdn.net/so/search?q=jenkins&spm=1001.2101.3001.7020)主目录下的config.xml文件以及jobs、users、workspace、plugins四个目录拷贝到新机器的jenkins主目录下。

docker cp /Users/panyurou/.jenkins/jobs  6257c7abfa11:/var/jenkins_home/jobs

docker cp /Users/panyurou/.jenkins/workspace  6257c7abfa11:/var/jenkins_home/jobs

# 2 增加前缀 /jenkins

docker run -d -p 8083:8080 --name jenkins -e JENKINS_CONTEXT_PATH=/jenkins jenkins/jenkins:2.346

# 3 没权限 

I have experienced this when the $JENKINS_HOME/jobs directory had incorrect permissions or ownership.

docker exec -it --user root 7666058a5629 bash

chown -R jenkins:jenkins /var/jenkins_home

# 4 stderr: No RSA host key is known for [xxx.xxx.xxx.xxx]:7999 and you have requested strict checking.

Manage Jenkins -> Security -> Configure Global Security

![image-20231003184529837](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231003184529837.png)

# 5 Docker部署jenkins配置公私钥拉取代码

https://blog.csdn.net/weixin_38080573/article/details/128482482

# 6 jdk，maven需要重新配置docker里安装的路径

maven settings.xml 仓库需要配置成docker里的仓库

# 7 设置开机自启docker

```
systemctl enable docker
```

# 8 设置docker启动时，启动jenkins容器

```shell
docker update --restart=always jenkins
```

