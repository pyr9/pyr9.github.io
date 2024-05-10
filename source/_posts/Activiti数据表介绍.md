---
title: Activiti——数据表介绍
date: 2023-02-02 21:55:15
tags:
categories: Activiti
---

Activiti 的表都以   ACT_   开头。 第二部分是表示表的用途的两个字母标识。 用途也和服务的 API 对应

![image-20230202215607751](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230202215607751.png)

### 表的命名规则和作用

- **ACT_RE** ：'RE'表示 repository。 这个前缀的表包含了流程定义和流程静态资源 （图片，规则，等等）。
- **ACT_RU**：'RU'表示 runtime。 这些运行时的表，包含流程实例，任务，变量，异步任务，等运行中的数据。 Activiti 只在流程实例执行过程中保存这些数据， 在流程结束时就会删除这些记录。 这样运行时表可以一直很小速度很快。
- **ACT_HI**：'HI'表示 history。 这些表包含历史数据，比如历史流程实例， 变量，任务等等。
- **ACT_GE** ： GE 表示 general。 通用数据， 用于不同场景下 

| **表分类**   | **表名**              | **解释**                                             |
| ------------ | --------------------- | ---------------------------------------------------- |
| 一般数据     |                       |                                                      |
|              | [ACT_GE_BYTEARRAY]    | 通用的流程定义和流程资源                             |
|              | [ACT_GE_PROPERTY]     | 系统相关属性                                         |
| 流程历史记录 |                       |                                                      |
|              | [ACT_HI_ACTINST]      | 历史的流程实例                                       |
|              | [ACT_HI_ATTACHMENT]   | 历史的流程附件                                       |
|              | [ACT_HI_COMMENT]      | 历史的说明性信息                                     |
|              | [ACT_HI_DETAIL]       | 历史的流程运行中的细节信息                           |
|              | [ACT_HI_IDENTITYLINK] | 历史的流程运行过程中用户关系                         |
|              | [ACT_HI_PROCINST]     | 历史的流程实例                                       |
|              | [ACT_HI_TASKINST]     | 历史的任务实例                                       |
|              | [ACT_HI_VARINST]      | 历史的流程运行中的变量信息                           |
| 流程定义表   |                       |                                                      |
|              | [ACT_RE_DEPLOYMENT]   | 部署单元信息                                         |
|              | [ACT_RE_MODEL]        | 模型信息                                             |
|              | [ACT_RE_PROCDEF]      | 已部署的流程定义 -> 同一个流程，部署一次增加一条记录 |
| 运行实例表   |                       |                                                      |
|              | [ACT_RU_EVENT_SUBSCR] | 运行时事件                                           |
|              | [ACT_RU_EXECUTION]    | 运行时流程执行实例                                   |
|              | [ACT_RU_IDENTITYLINK] | 运行时用户关系信息，存储任务节点与参与者的相关信息   |
|              | [ACT_RU_JOB]          | 运行时作业                                           |
|              | [ACT_RU_TASK]         | 运行时任务                                           |
|              | [ACT_RU_VARIABLE]     | 运行时变量表                                         |





![image-20230207214202834](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230207214202834.png)



下面的数据是：一个流程，部署后，开启了两个assginee不同的流程实例：

- act_re_deployment： 

![image-20230207221224629](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230207221224629.png)

- act_re_procdef：

![image-20230207221402715](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230207221402715.png)

- act_hi_procinst：

![image-20230207221437355](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230207221437355.png)

- act_hi_actinst：

![image-20230207221609189](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230207221609189.png)

- act_hi_taskinst： 

![image-20230207221658209](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230207221658209.png)

- act_hi_varinst：

![image-20230207221801474](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230207221801474.png)

- act_hi_identitylink：

![image-20230207221833964](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230207221833964.png)

- act_ru_task：

![image-20230207221951039](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230207221951039.png)
