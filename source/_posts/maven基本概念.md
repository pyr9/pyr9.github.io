---
title: maven基本概念
date: 2022-12-11 20:19:42
tags:
categories:
---

## maven遵循约定大于配置

- maven编译的路径为 src/main/java
- maven 打包后的路径在 /target/classes
- maven项目的配置文件存储在/resources目录下
- maven打包就是，运行`mvn package`把 /target/classes下的文件，打成一个jar包或者war包，打在target目录下
- maven运行测试用例，运行`mvn test`，将运行src/test下以test开后的文件里的，以test开头的方法、  如果引入了junit，将会运行以@test注解的方法。

## maven 生命周期

- Maven 的内部有三个标准生命周期
- 生命周期中的阶段是有严格的执行顺序的。比如：必须执行完compile才会执行package
- 生命周期中的阶段是有执行逻辑的。即当你执行一个阶段时，其前面的阶段会自动执行。
- 生命周期的阶段可以绑定具体的插件和目标

## Clean 生命周期

当我们执行 mvn post-clean 命令时，Maven 调用 clean 生命周期，它包含以下阶段：

- pre-clean：执行一些需要在clean之前完成的工作
- clean：移除所有上一次构建生成的文件
- post-clean：执行一些需要在clean之后立刻完成的工作

## Default (Build) 生命周期

这是 Maven 的主要生命周期，被用于构建应用，包括 23 个阶段，核心阶段包括：

| 核心阶段 | 详解                                                         |
| -------- | ------------------------------------------------------------ |
| validate | 验证项目是否正确，所有必要信息是否可用`（很少单独使用）`     |
| compile  | 编译项目的源代码（将src/main中的java代码编译成class文件，输出到targe目录下） |
| test     | 将单元测试的资源文件和代码进行编译，生成的文件位于target/test-classes （打包部署请跳过该阶段） |
| package  | 把class文件，resources文件打包成jar包（也可以是war包），生成的jar包位于target目录下 |
| verify   | 检查包是否有效`（很少单独使用）`                             |
| install  | 将jar部署到本地仓库，本地的其他模块依赖该jar包时，可以直接从本地仓库去获取 |
| deploy   | 将jar包部署到远端仓库，需要在maven的setting.xml中配置私服的用户名和密码，以及在pom.xml配置 |

 

![](https://panyuro.oss-cn-beijing.aliyuncs.com/20221227224007.png)

## Site 生命周期

Maven Site 插件一般用来创建新的报告文档、部署站点等。包括4各阶段

- pre-site：执行一些需要在生成站点文档之前完成的工作
- site：生成项目的站点文档
- post-site： 执行一些需要在生成站点文档之后完成的工作，并且为部署做准备
- site-deploy：将生成的站点文档部署到特定的服务器上