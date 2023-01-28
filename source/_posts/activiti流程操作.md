---
title: activiti流程操作
date: 2023-01-28 23:21:44
tags:
categories:
---

# 1 流程定义

使用idea插件设置流程图， 设置流程中需要的节点，节点负责人。具体步骤参考：

![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230128164658.png)

4 .流程部署

即：将流程图的内容存储到数据库

```java
package com.pyr;

import org.activiti.engine.ProcessEngine;
import org.activiti.engine.ProcessEngines;
import org.activiti.engine.RepositoryService;
import org.activiti.engine.repository.Deployment;
import org.junit.Test;

public class ActivitiDemo {

    @Test
    public void testDeployment(){
      // 1. 创建ProcessEngine
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
      // 2. 获取RepositoryService
        RepositoryService repositoryService = processEngine.getRepositoryService();
      // 3. 使用service进行流程的部署，定义流程名字，把bpmn部署到数据库中
        Deployment deploy = repositoryService.createDeployment()
                .name("话费报销")
                .addClasspathResource("bpmn/phoneExpense.bpmn20.xml")
                .deploy();
      // 4. 输出部署信息
        System.out.println("流程部署id="+deploy.getId());
        System.out.println("流程部署名字="+deploy.getName());
    }
}
```

![image-20230128210614102](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230128210614102.png)

流程定义部署后操作activiti的3张表如下：

- act_re_deployment 流程定义部署表，每部署一次增加一条记录

  ![](https://panyuro.oss-cn-beijing.aliyuncs.com/20230128204308.png)

- act_re_procdef 流程定义表，部署每个新的流程定义都会在这张表中增加一条记录。比如章三的话费报销是一条记录，李四的话费报销是一条记录。

  ![image-20230128204451742](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230128204451742.png) ![image-20230128204449359](/Users/panyurou/Library/Application Support/typora-user-images/image-20230128204449359.png)

- act_ge_bytearray 流程资源表

  ![image-20230128204546011](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230128204546011.png)

5. 启动流程实例

   ```java
       @Test
       public void testStartProcess(){
           // 1. 创建ProcessEngine
           ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
           // 2. 获取RuntimeService
           RuntimeService runtimeService = processEngine.getRuntimeService();
           // 3. 根据流程ID启动流程
           ProcessInstance instance = runtimeService.startProcessInstanceByKey("myProcess");
           // 4. 输出
           System.out.println("流程定义id="+instance.getProcessDefinitionId());
           System.out.println("流程实例id="+instance.getId());
           System.out.println("当前活动的id="+instance.getActivityId());
       }
   ```

   ![image-20230128210529180](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230128210529180.png)

    操作activiti的表如下：

​	     act_hi_actinst 流程实例执行历史

​		act_hi_identitylink 流程的参与用户历史信息

​		act_hi_procinst 流程实例历史信息

​		act_hi_taskinst 已经完成的任务信息

​		act_ru_execution 流程执行信息

​		act_ru_identitylink 流程的参与用户信息

​		act_ru_task 当前代办的任务信息

6. 查询自己所能处理的任务 

       ```java
    @Test
    public void testFindPersonalTasks(){
        // 1. 创建ProcessEngine
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        // 2. 获取taskService
        TaskService taskService = processEngine.getTaskService();
        // 3. 根据流程key和任务负责人 查询任务
        List<Task> tasks = taskService.createTaskQuery()
                .processDefinitionKey("myProcess")
                .taskAssignee("张三")
                .list();
        // 4. 输出
        tasks.forEach(task -> {
            System.out.println("流程实例id="+task.getProcessInstanceId());
            System.out.println("任务id="+task.getId());
            System.out.println("任务负责人="+task.getAssignee());
            System.out.println("任务名称="+task.getName());
        });
    }
       ```

![image-20230128210841640](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230128210841640.png)

7. 完成个人任务

   ```java
       @Test
       public void testCompleteTask(){
           // 1. 创建ProcessEngine
           ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
           // 2. 获取taskService
           TaskService taskService = processEngine.getTaskService();
           // 3. 根据任务id完成任务
           taskService.complete("2505");
       }
   ```

   操作activiti的表如下：

   act_hi_taskinst:  已经完成的历史任务信息 -> 插入新数据; 更新完成任务的结束时间

   ![image-20230128213152660](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230128213152660.png)

   act_ru_task：当前代办的任务信息 -> 删除旧数据，插入新数据

   ![image-20230128213015838](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230128213015838.png)

​       act_hi_identitylink: 流程的参与用户历史信息  -> 插入新数据

![image-20230128213626199](/Users/panyurou/Library/Application Support/typora-user-images/image-20230128213626199.png)

​		  

   8. 更加灵活的完成个人任务

      ```java
          @Test
          public void testCompleteTask2(){
              // 1. 创建ProcessEngine
              ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
              // 2. 获取taskService
              TaskService taskService = processEngine.getTaskService();
              // 3. 如果可以确定是一个任务，可以直接通过singleResult获得，依次修改taskAssignee为直线经理，部门经理，财务人员
              Task task = taskService.createTaskQuery()
                      .processDefinitionKey("myProcess")
                      .taskAssignee("财务人员")
                      .singleResult();
              // 3. 根据任务id完成任务
              taskService.complete(task.getId());
              System.out.println("DONE!");
          }
      ```

      此时查看act_ru_task表，已经没有数据了，因为任务执行完了

      查看act_hi_actinst，可以看到END_TIME都已经填充上了

      ![image-20230128231354324](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230128231354324.png)
