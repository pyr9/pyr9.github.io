---
title: Spring Data JPA 使用QueryDSL
date: 2022-10-20 21:52:08
tags:
- springdata jpa
- queryDSL
categories: JPA
---

# 1 QueryDSL 是什么？

- QueryDSL是基于ORM框架或SQL平台上的**一个通用查询框架**。借助QueryDSL可以在任何支持的ORM框架或SQL平台 **上以通用API方式构建查询。**
- JPA是QueryDSL的主要集成技术，是JPQL和Criteria查询的代替方法。目前QueryDSL支持的平台包括 JPA,JDO,SQL,Mongodb 等等

- Querydsl扩展能让我们以**链式方式代码编写查询方法**。该扩展需要一个接口QueryDslPredicateExecutor，它定义了很多查询方法。

# 2 Spring Data JPA 使用QueryDSL

## 1. 引入依赖

```java
    <dependencies>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.8</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.querydsl/querydsl-jpa -->
        <dependency>
            <groupId>com.querydsl</groupId>
            <artifactId>querydsl-jpa</artifactId>
            <version>4.1.4</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.3.19</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.mysema.maven</groupId>
            <artifactId>apt-maven-plugin</artifactId>
            <version>1.1.3</version>
        </dependency>
        <dependency>
            <groupId>com.querydsl</groupId>
            <artifactId>querydsl-apt</artifactId>
            <version>4.2.1</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
<!--  Spring不同模块或者与外部进行集成时，依赖处理就需要各自对应版本号。BOM是由Maven提供的功能，用以统一间接或者直接依赖的类库版本-->
<!--    在maven的pom.xml中无需指定具体的类库版本，直接使用，即默认使用bom中指定的版本
<dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.data</groupId>
                <artifactId>spring-data-bom</artifactId>
                <version>2021.1.0</version>
                <scope>import</scope>
                <type>pom</type>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <build>
        <plugins>
            <!--该插件可以生成querysdl需要的查询对象(命名方式为："Q"+对应实体名，执行mvn compile即可-->
            <plugin>
                <groupId>com.mysema.maven</groupId>
                <artifactId>apt-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>process</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>target/generated‐sources/queries</outputDirectory>
                            <processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
                            <logOnlyOnError>true</logOnlyOnError>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```



## 2 . 添加实体类和Jpa的配置

参考：[Spring data JPA的CRUD - 楼上有只喵 (pyr9.github.io)](https://pyr9.github.io/Spring data JPA的CRUD/)

## 3. 编写Repository

```java
package org.pyr.repositories;

import org.pyr.entity.Customer;
import org.springframework.data.querydsl.QuerydslPredicateExecutor;
import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface CustomerRepository extends CrudRepository<Customer, Long> , QuerydslPredicateExecutor<Customer> {
}
```

## 4 编写测试类

```java
package org.pry;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.pyr.config.JPAConfig;
import org.pyr.entity.Customer;
import org.pyr.entity.QCustomer;
import org.pyr.repositories.CustomerRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@ContextConfiguration(classes = JPAConfig.class)
@RunWith(SpringJUnit4ClassRunner.class)
public class QueryDslTest {
    @Autowired
    CustomerRepository customerRepository;

    @Test
    public void testFindOne() {
        QCustomer qCustomer = QCustomer.customer;
        Customer customer = customerRepository.findOne(
                qCustomer.custId.eq(4L)
                        .and(qCustomer.custName.eq("aa"))
        ).orElse(null);
        System.out.println(customer);
    }
    @Test
    public void testFindAll() {
        QCustomer qCustomer = QCustomer.customer;
        Iterable<Customer> customers = customerRepository.findAll(
                qCustomer.custId.in(4L, 5L, 6L, 7L)
                        .and(qCustomer.custName.eq("aa"))
        );
        System.out.println(customers);
    }
}
```



