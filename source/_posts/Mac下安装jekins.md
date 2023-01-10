---
title: Mac下安装jekins
date: 2023-01-10 22:04:19
tags:
categories:
---

用brew 安装

` brew install jenkins`

启动jenkins

`brew services start jenkins`

停止jenkins

` brew services stop jenkins`

# 修改jenkins默认端口配置

## 1.查看jenkins安装路径

-  `brew list jenkins`

<img src="https://panyuro.oss-cn-beijing.aliyuncs.com/20230110221220.png" style="zoom:150%;" />

## 2. 修改homebrew.mxcl.jenkins.plist

- 打开文件 `vim /opt/homebrew/Cellar/jenkins/2.385/homebrew.mxcl.jenkins.plist`

- 修改默认端口号

  ![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230110221801.png)

## 3. 修改jenkins系统配置url

<img src="https://panyuro.oss-cn-beijing.aliyuncs.com/20230110222629.png"  />

<img src="/Users/panyurou/Library/Application Support/typora-user-images/image-20230110222115537.png" alt="image-20230110222115537" style="zoom:86%;" />
