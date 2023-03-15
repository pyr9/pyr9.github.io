---
title: jenkins部署vue项目
date: 2023-03-15 14:44:30
tags: 
categories: jenkins
---

# 1 配置nodeJs

因为Jenkins容器中只有java环境支持运行jenkins，没有node环境，但是jenkins提供在线安装nodejs。[官方文档](https://plugins.jenkins.io/nodejs)

1. 系统管理--->管理插件--->下载NodeJS插件

![image-20230315155958382](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230315155958382.png)

2. 系统管理--->全局工具配置--->选择需要安装的nodejs版本

Jenkins 会从nodejs官网下载安装，nodejs安装包在：$JENKINS_HOME/tools目录下

![image-20230315155909887](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230315155909887.png)



# 2 新建一个任务

![image-20230315160310957](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230315160310957.png)



# 3 配置仓库地址和分支名

![image-20230315160535916](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230315160535916.png)

# 4 配置构建环境

##### 构建环境勾选 Provide Node & npm bin/folder to PATH

- 每次build，都会首先执行环境构建，环境构建无误后，才会开始真正的构建过程
- 会下载nodejs并安装配置，并把node添加到当前PATH环境变量中，这样就支持node和npm命令了

![image-20230315160620905](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230315160620905.png)

#  5 配置shell

构建中打印$PATH并查看node，npm版本

![image-20230315161043379](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230315161043379.png)



至此，可以点击立即构建了
