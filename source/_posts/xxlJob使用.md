---
title: xxlJob使用
date: 2023-11-14 14:29:20
tags:
categories:  
---

# 1. 任务调度

## 1. 什么是任务调度

任务调度是为了自动完成特定任务，在约定的特定时刻去执行任务的过程。

## 2. 什么场景会需要任务调度

- 某电商平台需要每天上午10点，下午3点，晚上8点发放一批优惠券
- 某银行系统需要在信用卡到期还款日的前3天进行短信提醒
- 某财务系统需要每天凌晨结算前一天的财务数据

# 2. 分布式任务调度

## 1. 什么是分布式任务调度

采用分布式架构，一个服务往往会部署多个冗余实例来运行我们的业务，在这种分布式系统环境下运行任务调度，我们称之为**分布式任务调度**

## 2. 为什么需要分布式任务调度

使用spring提供的@scheduled，也能实现调度的功能，为什么还需要分布式任务调度的呢？

- 高可用：单机版的任务调度只能在一台机器上运行，程序出现异常后会导致服务不可用。
- 防止任务重复执行。当部署了多台服务，同一个定时任务需要控制在同一时间内，只有一个任务在执行。
- 单机处理存在极限。即使可以通过多线程提高单机的处理效率但能力依旧有限（CPU，内存，硬盘），依旧存在处理不过来的场景

# 2. xxl-Job

## 1. 定义

xxlJob是一个轻量级**分布式任务调度**平台

## 2. 设计思想

- 将调度行为抽象形成“调度中心”公共平台，而平台自身并不承担业务逻辑，“调度中心”负责发起调度请求。
- 将任务抽象成分散的JobHandler，交由“执行器”统一管理，“执行器”负责接收调度请求并执行对应的JobHandler中业务逻辑。
- 因此，“调度”和“任务”两部分可以相互解耦，提高系统整体稳定性和扩展性；

## 3. 系统组成

- **调度模块（调度中心）**：

  负责管理调度信息，按照调度配置发出调度请求，自身不承担业务代码。

  调度系统与任务解耦，提高了系统可用性和稳定性，同时调度系统性能不再受限于任务模块；
  支持可视化、简单且动态的管理调度信息，包括任务新建，更新，删除，GLUE开发和任务报警等，所有上述操作都会实时生效，同时支持监控调度结果以及执行日志，支持执行器Failover。

- **执行模块（执行器）**：
  负责接收调度请求并执行任务逻辑。任务模块专注于任务的执行等操作，开发和维护更加简单和高效；
  接收“调度中心”的执行请求、终止请求和日志请求等。

## 4 **Spring Boot 集成 XXL-JOB**

参考[快速入门](https://www.xuxueli.com/xxl-job/#%E4%BA%8C%E3%80%81%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8)

 ### 1. 配置运行调度中心（xxl-job-admin）

1. 首先从源码仓库中下载代码
2. 执行doc/db` 目录下有数据库脚本 `tables_xxl_job.sql，初始化调度数据库 `xxl_job`
3. 修改 xxl-job-admin 的配置文件，主要是修改数据源信息，若需要用到邮件报警功能，需要配置邮箱
4. 启动XxlJobAdminApplication，正常启动后，访问地址为：`http://localhost:8080/xxl-job-admin`，默认的账户为 admin，密码为 123456，

### 2. 配置运行执行器项目（xxl-job-executor）

1. 添加依赖

   ```xml
   implementation 'com.xuxueli:xxl-job-core:2.3.0'
   ```

2. 加入配置

```properties
### 调度中心部署跟地址 [选填]：如调度中心集群部署存在多个地址则用逗号分隔。执行器将会使用该地址进行"执行器心跳注册"和"任务结果回调"；为空则关闭自动注册；
xxl.job.admin.addresses=http://localhost:6100/xxl-job-admin
### 执行器通讯TOKEN [选填]：非空时启用；
xxl.job.accessToken=
### 执行器AppName [选填]：执行器心跳注册分组依据；为空则关闭自动注册
xxl.job.executor.appname=xxl-job-executor-city-eureka
### 执行器注册 [选填]：优先使用该配置作为注册地址，为空时使用内嵌服务 ”IP:PORT“ 作为注册地址。从而更灵活的支持容器类型执行器动态IP和动态映射端口问题。
xxl.job.executor.address=
### 执行器IP [选填]：默认为空表示自动获取IP，多网卡时可手动设置指定IP，该IP不会绑定Host仅作为通讯时用；地址信息用于 "执行器注册" 和 "调度中心请求并触发任务"；
xxl.job.executor.ip=localhost
### 执行器端口号 [选填]：小于等于0则自动获取；默认端口为9999，单机部署多个执行器时，注意要配置不同执行器端口；
xxl.job.executor.port=9999
### 执行器运行日志文件存储磁盘路径 [选填] ：需要对该路径拥有读写权限；为空则使用默认路径；
xxl.job.executor.logpath=/data/applogs/xxl-job/jobhandler
### 执行器日志文件保存天数 [选填] ： 过期日志自动清理, 限制值大于等于3时生效; 否则, 如-1, 关闭自动清理功能；
xxl.job.executor.logretentiondays=30
```

3. 在 config 包下创建 `XxlJobConfig` 类

```java
package org.pyr.city.config;

import com.xxl.job.core.executor.impl.XxlJobSpringExecutor;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * xxl-job config
 *
 * @author xuxueli 2017-04-28
 */
@Configuration
public class XxlJobConfig {
    private Logger logger = LoggerFactory.getLogger(XxlJobConfig.class);

    @Value("${xxl.job.admin.addresses}")
    private String adminAddresses;

    @Value("${xxl.job.accessToken}")
    private String accessToken;

    @Value("${xxl.job.executor.appname}")
    private String appname;

    @Value("${xxl.job.executor.address}")
    private String address;

    @Value("${xxl.job.executor.ip}")
    private String ip;

    @Value("${xxl.job.executor.port}")
    private int port;

    @Value("${xxl.job.executor.logpath}")
    private String logPath;

    @Value("${xxl.job.executor.logretentiondays}")
    private int logRetentionDays;


    @Bean
    public XxlJobSpringExecutor xxlJobExecutor() {
        logger.info(">>>>>>>>>>> xxl-job config init.");
        XxlJobSpringExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();
        xxlJobSpringExecutor.setAdminAddresses(adminAddresses);
        xxlJobSpringExecutor.setAppname(appname);
        xxlJobSpringExecutor.setAddress(address);
        xxlJobSpringExecutor.setIp(ip);
        xxlJobSpringExecutor.setPort(port);
        xxlJobSpringExecutor.setAccessToken(accessToken);
        xxlJobSpringExecutor.setLogPath(logPath);
        xxlJobSpringExecutor.setLogRetentionDays(logRetentionDays);

        return xxlJobSpringExecutor;
    }
}
```

4. 编写 JobHandler

```java
@Component
public class SimpleXxlJob {
    @XxlJob("demoJobHandler")
    public void demoJobHandler(){
        System.out.println("执行定时任务，执行时间为："+new Date());
    }
}
```

### 3. BEAN模式测试

BEAN模式执行器：每个执行器都是Spring的一个Bean实例，XXL-JOB通过注解 `@XxlJob`的方式进行任务开发；

1. 编写 JobHandler

```java
@Component
public class SimpleXxlJob {
    @XxlJob("demoJobHandler")
    public void demoJobHandler(){
        System.out.println("执行定时任务，执行时间为："+new Date());
    }
}
```

2. 在执行器管理页面添加该执行器。

![image-20231114165019091](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231114165019091.png)

3. 在任务管理界面添加我们刚才开发的任务，运行模式选择BEAN

![image-20231114165049371](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231114165049371.png)

4. 在操作页面，点击执行一次

![image-20231114165154323](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231114165154323.png)

### 4. GLUE(Java)模式测试

1. 在任务管理界面添加一个新任务，运行模式选择BEAN

![image-20231114183450401](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231114183450401.png)

2. 编辑需要执行的代码

![image-20231114183527467](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231114183527467.png)

![image-20231114183541042](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231114183541042.png)

3. 在操作页面，点击执行一次

![image-20231114183617278](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231114183617278.png)

5. 执行日志打印

需要使用`XxlJobHelper.log`打印执行日志；

![image-20231114184326029](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231114184326029.png)

![image-20231114184858772](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231114184858772.png)

![image-20231114184353146](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231114184353146.png)
