---
title: MES系统后台设计
date: 2024-01-24 14:57:26
tags:
password: 19980617pyr
categories: 项目
---

# 1. 什么是MES

这个是给葫芦岛做的一个工业数字化项目，从19年开始做的，主要包括排产，供应链，物流，设备，数采等多个模块。

是一个前后端分离项目，前端是用的vue2，后端是SpringCloud Netflix那一套：注册中心eureka， 网关：spring cloud gateway, 服务调用：Feign，使用springdata Jpa做持久层框架，数据库使用的是pgsql，influxDb，消息中间件是：rabbitMQ， 定时任务：xxlJob, 工作流：activiti6 ,  使用docker部署jenkins和elk, nginx等中间件。

我在其中主要开发的模块是用户和采购模块以及运维



# 2. 业务模块概述

1. **生产计划：** APS: 高级计划排产。 包括项目计划，工单，工序。APS系统可以基于生产需求、资源约束、库存水平等因素，自动生成最优的生产计划，以实现生产资源的有效利用和生产成本的最小化。
1. **供应链管理：** 包括PR（PR需求，报价，询价），PO（全部PO，PO技术澄清），供应商管理，发货清单，发货任务，外检管理
2. **物流管理：**物流到货管理，异常到货管理（发起，待办，列表）、物料配送
3. **质量管理：** 105入场检验，106退货，产品终检，退库检验，量器具管理
4. **设备管理：** 设备档案、设备维修、设备故障管理、设备状态
5. **看板**：数据的整理和以图表的方式展示出来
6. **车床数采：**使用opc采集主轴数据（倍率，负载，运转状态，进给速度，进给倍率）

# 3. 模块实战

## 1. 供应链模块SCM

供应链管理（Supply Chain Management, SCM）是一个复杂且多层次的过程，涵盖了从原材料采购到最终产品交付给客户的整个流程。以下是SCM的全流程概述，

对接SAP系统

- SAP人：SAP每天导出PR(采购申请)
- 开发人员：对接SAP，录入系统
- 业务方：PR界面有具体的项目号，WBS号，PR号，物料号，描述等，因此可以对具体的PR指定供应商询价
- 供应商：进入【供应商报价管理】对PR报价
- 业务方：在【供应商报价管理】查询具体报价金额， 下订单PO到SAP
-  开发人员：对接SAP，将PO录入系统
- 根据PO，创建发货清单， 生成发货任务

## 2. 物流模块VMS

- 物料到货管理：po, po_item, 物料号，项目名称，物料描述，物流状态
- 异常到货管理
- 101，102，103，104，105，106

## 3. 质量管理QMS

确保产品和服务的质量符合标准和客户需求。

1. 入厂检验
2. 过程检验
3. 产品终检
4. 退货
5. 库房管理：转库、移库（物料号，源BIN位、目的BIN位）

## 4. 设备管理TPM

- 设备基础数据：名称，编码，等级， 制造商
- 设备维修，点巡检（定期检查设备的关键部位），设备保养，设备故障  ——》（工作流）
- 设备状态采集：通过OPC调用获取到不同节点的数据，收集展示出来。如不同时间节点上的设备主轴数据：倍率，负载，运转状态，进给速度，进给倍率来确保设备状态

## 5. 高级计划排产APS

- 项目计划
- 主计划排产：根据计划开始时间和计划完成时间，以及节假日
- 工单工序等

## 6. 通用模块

- 用户模块

# 4. 业务点

### 1. 菜单权限动态配置

## 1. 用户对于数据的增删改查权限的设计

我们目前是基于`RBAC0`， 最基本的模型，未做特殊要求和扩展，在该模型中，一个用户可以同时拥有多个角色，每个角色可以拥有多个权限。

具体实现上：

1. 配置菜单

   1. 不同页面的前端router地址

   基于tree的结构，配置不同界面的地址，菜单名称，中英文以及菜单类型（list/edit/todo）

2. 创建角色，设置角色名称， 角色读写权限等级（增/删/改/查）中的组合。给这个权限配置新建的权限点以及对应的人

前端上：

在进入页面的时候，获取用户可以访问的所有菜单地址，前端根据菜单结构绘制出左侧导航栏。

在前端界面缓存这样一份数据，通过类似拦截器的功能，如果用户直接输入这个前端界面地址，放回403

权限的获取和拦截

（增删改查，分别对应2，3，5，7 这样的话，数据库只需要存一位去记录，当然存0101这种也行）

1. 在权限拦截器增加校验
   1. 获取当前登录用户可以访问的权限列表
   2. 将权限列表和当前请求做比较，需满足以下两个条件，即可放行，否则提示没有权限
      - 请求URL在当前权限URL列表里
   3. 当前访问的method也在权限URL列表里（这里需要后端使用restful请求，即增删改查对应 POST，DELETE， PUT，GET， 或者后端带标识过来）



## 2. 引入用户组的概念，对数据的查询权限进行控制

需求：不同供应商只能看到自己的数据

用户数量比较多，一个个加角色工作量比较大，而且数据权限也不好控制。可以加入用户组模式。需要给用户分组，每个用户组内有多个用户，可以给用户授权外，也可以给用户组授权。最终用户拥有的所有权限 = 用户个人拥有的权限+该用户所在用户组拥有的权限。

类似 svn 中的用户权限，比如，将一个svn用户加入到 group中，然后设置group的权限，以后加入更多的用户，就不用再一一设置用户的权限了



引入用户组的概念：

- 将不同的供应商建立成不同的用户组

- 给用户组加角色（供应商） 

- - 查询用户角色的时候+用户组的角色

- - 查询用户权限+用户组的角色上创建的权限

- 用户组加子账户 

- - 在供应商查询的界面：比如物料报价界面，采购订单界面，对应的entity，都存储上user_group 字段

- - 查询数据的时候，根据用户组过滤，当前用户所在用户组和数据上存储的用户组一致

### 2. 用户鉴权

引入springCloud 网关，在后端对接口进行统一校验

### 1. 用户登录的实现

1. 登录页面重构成前后端分离，（引入了springCloud gateway）
   1. 用户输入账户密码，手机号验证码，点击提交
   2. 后端根据用户密码(数据库存储的是MD5加密过的密码)去用户表里查询是否有当前用户
   3. 存在当前用户，则检查验证码是否匹配
   4. 匹配则创建以及保存jwt, 将jwt返回给前端
   5. 前端将jwt存储在localStorage里
   6. 前端调用axios请求时，将jwt带进request header 里的【Authorization】
   7. 后端在common 模块里，增加拦截器，从request取出【Authorization】，解析jwt为current User，每个引入了common模块的微服务自己分别将jwt解析成currentUser,  并将其存储在TransmittableThreadLocal里
   8. 权限拦截器，对当前currentUser判断是否有当前url的访问权限
   9. 后面在当前微服务中通过feign或者restTemplate去请求其他微服务接口时，分别通过增加feign拦截器和restTemplate拦截器，对jwt进行

  加入了网关：

       1. 从第7步开始更新，请求到了后端后，全进入网关，后端网关接收到jwt, 在网关拦截器解析成了currentUser，放到request的header里   URLEncoder.encode(EmJsonUtils.objectToJson(EmRequestHolder.getCurrentUser()), UTF_8)
       2. 在common模块增加拦截器，解析header获取到current User
       3. 当前微服务中通过feign或者restTemplate去请求其他微服务接口时，分别通过增加feign拦截器和restTemplate拦截器，将currentUser 放到header里（因为之前的逻辑是从request的header里取【Authorization】，解析jwt，现在网关都解析过了，发给微服务的请求里已经没有【Authorization】，只有解析过的currentUser）

**网关带来的其他好处：**

1. 之前URL权限检验的时候，某些URL需要设置成白名单，最初是在每个微服务里通过common模块去check权限的，就需要在每个微服务模块里都配置白名单的URL列表，现在改到网关里后只需要在网关配置即可
2. 老模式：将jwt放到了admin-common里，每个微服务都需要自己鉴权，处理多次，有性能消耗
   3. 之前客户端需要知道每个微服务的具体端口，现在只需要知道网关即可（也就是nginx的配置）

## 



## 3. 文件批量上传附件

附件上传时，支持预览，解决pdf xss安全问题

选中一条记录时，允许删除已经上传的附件 选中多条记录只允许在现有基础上增加附件，因为批量上传的时候选中多条记录，前端无法确定是回显哪一条记录的已有附件，故无法进行删除操作。

1. 点击按钮，打开上传附件，`/emAttachment/upload`,  接收MultipartFile。进行check文件大小/类型，文件加密

2. 根据加密的结果，在服务器上查找文件是否已经存在，避免重复的文件多次上传，占用服务器空间。不存在创建新文件，存在直接获取之前的attachment。上传成功后，此时attachment表里，对应字段fileName，md5File， fileSize， extName。

3. 点击保存按钮，调用batchUpload接口。将此次新增的附件ID和原始的ID拼接起来，通过反射，设置到这个entity的指定字段上。另外更新附件表里，对应的linkedTableName，linkedFieldName，linkedTableRecordId。

   ```java
   public class UploadAttachmentForm {
       List<Long> entityIds; // 哪些条目需要上传附件
       String attachmentIds; // 需要上传的附件IDS
       String attachmentFieldName; // 需要更新到哪个字段上
   }
   ```

- 

4. 批量上传接口上，增加注解NeedUpdateAttachment，以及指明【存储附件id的字段名】

5. 实体类上，存储附件id的字段，增加注解@hasRelatedAttachment，后续切面需要根据该字段，在上传附件和删除数据时，进行附件的更新和删除 

   - 在删除entity的逻辑上，增加注解＠NeedDeleteAttachment，删除引用次数

   - 在上传附件的逻辑上，增加注解＠NeedUpdateAttachment，增加引用次数

6. 在查询的接口，需要返回附件信息的，增加NeedFillAttachment，通过获取entity，再拿到这个entity上，有hasRelatedAttachment注解的字段，从而在附件表查询到具体的entity属性　

> 附件上记录引用次数attachment.setReferencedCount，因为存在多个entity在服务器上对应同样的附件，那么每一个entity的使用都需要增加或减少引用count，当count为０时，则删除entity时也删除附件

**另外**：

- 需要新建定时任务，删除没有linked_record的记录。
- 根据业务模块设置可以上传的最大附件大小

## 9. 批量生成SQL

### 1  背景：

JPA的JpaRepository的saveAll方法，底层是foreach 的save去循环调用的，对于更新和新增性能很低。

### 2. 优化：

step1: 先配置了批处理，即打包10个SQl一起执行一次

step2：自定义工具类生成insert/update语句

```java
insert: "insert into %s (%s) values %s;"
```

```java
update %s set %s from (values %s) as tmp (%s) where %s.id=tmp.id;

eg:
UPDATE sys_user
SET user_name = tmp.new_username,
telephone = tmp.new_telephone
FROM ( VALUES
(1642898053143801857, 'Smith', 13222228888),
(1642898650068758530, 'Johnson',15222299999)
) AS tmp(id, new_username, new_telephone)
WHERE sys_user.id = tmp.id;
```

注意：

1. 避免SQL语句过长超过数据库限制，可以根据指定长度对SQL进行拆分，每次往SQL上拼接时，去

检查下有没有超过最大长度，37MB的sql都没问题，象征性限制了10M

2. 生成可用ID范围

新建一张表，来存储当前生成的ID，

根据业务，表名对应查询当前ID的值，根据当前ID再加上此次需要保存的entity数量，生成ID范围

列表，再设置到entity的ID上，生成SQl，可以使用CAS的思想，避免ID重复

# 运维相关

## 1. 引入jenkins提高团队部署效率

1. git clone -> maven clean install -> jar 包推送到客户服务器 （jar包带有当前日期时间戳，方便后期回滚）-> 执行kill -9 & nohub java -jar -> 飞书webhook通知          
2. jenkins每个开发者使用自己的账号，这样每次在群里就可以看到当前是否有人在升级，以及升级结果如何
3. Jenkins 回滚机制：目前是寻找一天前，最新的jar包重新部署                                                                                                                                                                                                                                                                                                                                                                   

## 2. 引入ELK提高排查日志的速度

- 使用filebeat去收集日志文件，log stash对日志进行了过滤，以及统一格式，存储在ES中，使用kabana搜索
- 索引占用空间过大，设置索引定期清除策略

## 3. 引入docker, 简化中间件的管理与迁移

- Docker-compose对Java服务和中间件进行统一配置，后续在本地只需要安装docker , 执行docker compose up 
- 在docker-compose设置服务，电脑重启后自动重启，避免之前手动重启
