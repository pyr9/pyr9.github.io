---
title: Activiti——分配任务负责人
date: 2023-02-09 23:33:18
tags:
categories: Activiti
---

# 1 固定分配

在进行业务流程建模时指定固定的任务负责人， 如图：

![image-20230210163130769](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230210163130769.png)

# 2 表达式分配

Activiti 使用 UEL 表达式， UEL 是 java EE6 规范的一部分， UEL（Unified Expression Language）即 统一表达式语言， activiti 支持两个 UEL 表达式： UEL-value 和 UEL-method。 

## 2.1  UEL-value

- assignee 这个变量是 activiti 的一个流程变量，

![image-20230210163518444](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230210163518444.png)

- user 也是 activiti 的一个流程变量， user.name 表示通过调用 user 的 getter 方法获取值

![image-20230210163909807](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230210163909807.png)

## 2.2  UEL-method

userBean 是 spring 容器中的一个 bean，表示调用该 bean 的 getUserId()方法。 

![image-20230210164303115](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230210164303115.png)

## 2.3 其他

表达式支持解析基础类型、 bean、 list、 array 和 map，也可作为条件判断。如下：
${order.price > 100 && order.price < 250} 

# 3 监听器分配

可以使用监听器来完成很多Activiti流程的业务。

任务监听器是发生对应的任务相关事件时执行自定义 java 逻辑 或表达式。

任务相当事件包括：  

![1577506842889](https://panyuro.oss-cn-beijing.aliyuncs.com/1577506842889.png)
