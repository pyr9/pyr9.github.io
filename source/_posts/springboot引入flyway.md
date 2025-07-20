---
title: springboot引入flyway
date: 2023-11-09 15:20:57
tags:
categories: Springboot
---

# 1. flyway 是什么？

Flyway 就是一款数据库界的版本控制工具，它可以记录数据库的变化记录

**为什么需要它？**

目前通过人工去维护、同步数据库脚本，但经常会遇到疏忽而遗漏的情况，比如我们在开发环境对某个表新增了一个字段，而部署到线上时却忘了执行该 SQL 脚本，导致出现 bug。

有了 Flyway，在Spring boot项目启动时，会自动执行flyway定义的 SQL ，而无需人为手工控制，再也不用担心因数据库不同步而导致的各种环境问题。

# 2. springboot引入flyway

## 1. 引入依赖

pom.xml增加

```xml
        <dependency>
            <groupId>org.flywaydb</groupId>
            <artifactId>flyway-core</artifactId>
        </dependency>
```

## 2. 支持maven编译sql文件(根据情况，可以不配)

pom.xml增加

```xml
      <resources>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.sql</include>
                </includes>
            </resource>
        </resources>
```

## 2. 添加配置

application.yml增加

```yml
spring:
  flyway:
    enabled: true ## 开启flyway, sithMesDev不开启为false
    baseline-on-migrate: true #对于已经存在的项目，数据库中存在数据，通过设置baseline告诉flyway，这个baseline及之前的sql脚本都不要执行了
    locations:
      - classpath:db.migration #指定项目中需要迁移的sql文件在哪个位置，在springboot项目中classpath指的是resource文件夹
    table: flyway_schema_history_xxx # 版本控制日志表，默认flyway_schema_history,不同系统建议修改表名，如flyway_schema_history_admin
```

## 3. 创建迁移所需要的脚本

在项目src/main/resources下新建文件夹层级db/migration，将需要执行的sql文件放进去

![image-20231109154251326](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231109154251326.png)



## 4 测试

重启项目，可以观察控制会出现：

![image-20231109154142159](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20231109154142159.png)





