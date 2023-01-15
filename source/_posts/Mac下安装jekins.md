---
title: jekins入门
date: 2023-01-10 22:04:19
tags:
categories: jenkins
---

# 1. 可持续化集成CI

​      持续集成即CI（Continuous integration）是一种软件开发实践,可以让团队在持续的基础上不断收到反馈并进行改进，不必等到开发周期后期才寻找缺陷。持续集成要点：

- 统一的代码库 git
- 统一的依赖包管理 nexus
- 测试自动化
- 构建全自动化 maven
- 部署自动化
- 可追踪的集成记录 ：某一次有问题，可以找到上次集成。或者上上次集成，然后代码回滚

持续集成的流程：

![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230108202610.png)

# 2 jenkins概述

jenkins就是为了满足上述持续集成的要点而设计的一款工具，主体框架是采用java开发，实质内部功能都是通过插件实现，极大程度提高了系统的扩展性。

其不仅可以满足java系统的集成，也可以实现PHP等语言的集成发布。

通过pipeline插件，用户可以随自己需要定制集成流程。

# 3 mac 下安装 jenkins

用brew 安装

` brew install jenkins`

启动jenkins

`brew services start jenkins`

停止jenkins

` brew services stop jenkins`

# 4 修改jenkins默认端口配置

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
