---
title: JPA和hibernate和SpringDataJpa
date: 2022-10-16 16:03:46
tags: JPA
categories: JPA
---

Java访问持久层的两种方式

- 以Sql为核心，封装一定程度的JDBC操作，比如mybatis
- 以Java实体类为核心，将实体类和数据表之间建立连接，也就是我们说的ORM框架，比如：Hibernate,SpringData jpa

# 1. JPA

JPA顾名思义就是Java Persistence API的意思

- 使用注解或XML描述对象－关系表的映射关系
- 将运行期的实体对象持久化到数据库中
- 定义了一些接口。

# 2. hibernate

Hibernate就是实现了JPA接口的ORM框架。

# 3 **Spring Data JPA**

## 3.1 定义

- Spring Data JPA是spring提供的一套 **JPA 应用框架**。

- SpringDataJpa可以理解为JPA规范的再次封装抽象，底层还是使用了Hibernate的Jpa技术实现。

- Spring Data JPA旨在通过将统一ORM框架的访问持久层的操作，来提高开发人的效率。

- 虽然ORM框架都实现了JPA规范，但是在不同的ORM框架之间切换仍然需要编写不同的代码，而使用**Spring Data JPA能够方便大家在不同的ORM框架之间进行切换而不需要更改代码**。

- 按照约定好的规则进行【方法命名】去写dao层接口，就可以在不写接口实现的情况下，实现对数据库的访问和操作。同时提供了很多除了CRUD之外的功能，如分页、排序、复杂查询等等

- Spring Data JPA 让我们解脱了DAO层的操作，基本上所有CRUD都可以依赖于它来实现,

  

## 3.2  **Spring Data JPA给我们提供的主要的类和接口**

- Repository 接口：

  - Repository

  - CrudRepository

  - JpaRepository = 基本`CRUD`功能+分页+按“实例”查询

    ```java
    public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor
    ```

- Repository 实现类：

  - SimpleJpaRepository

  - QueryDslJpaRepository

# 4 JPA、hibernate、Spring Data Jpa关系

- JPA是一个规范，也就是说它仅仅定义了一些接口
- Hibernate就是实现了JPA接口的ORM框架

- SpringDataJpa是Spring提供的一套简化JPA开发的框架。可以理解为JPA规范的再次封装抽象，底层还是使用了Hibernate的Jpa技术实现。

![image-20230320175230013](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230320175230013.png)

# 5 **Spring Data JPA和Hibernate的关系**

- Hibernate其实是JPA的一种实现，而Spring Data JPA是一个JPA数据访问抽象。
- pring Data JPA不是一个实现或JPA提供的程序，它只是一个抽象层，主要用于减少为各种持久层存储实现数据访问层所需的样板代码量。但是它还是需要JPA提供实现程序。
- Spring Data JPA是一种JPA的抽象层，底层依赖Hibernate

# 6 mybatis 和hibernate的对比？

（1）hibernate是全自动，而mybatis是半自动。

hibernate完全可以通过对象关系模型实现对数据库的操作，拥有完整的JavaBean对象与数据库的映射结构来自动生成sql。而mybatis仅有基本的字段映射，对象数据以及对象实际关系仍然需要通过手写sql来实现和管理。

（2）hibernate数据库移植性远大于mybatis

hibernate通过它强大的映射结构和hql语言，大大降低了对象与数据库（Oracle、MySQL等）的耦合性，而mybatis由于需要手写sql，因此与数据库的耦合性直接取决于程序员写sql的方法，如果sql不具通用性而用了很多某数据库特性的sql语句的话，移植性也会随之降低很多，成本很高。

（3）hibernate拥有完整的日志系统，mybatis则欠缺一些

hibernate日志系统非常健全，涉及广泛，包括：sql记录、关系异常、优化警告、缓存提示、脏数据警告等；而mybatis则除了基本记录功能外，功能薄弱很多。

（4）sql直接优化上，mybatis要比hibernate方便很多

由于mybatis的sql都是写在xml里，因此优化sql比hibernate方便很多。而hibernate的sql很多都是自动生成的，无法直接维护sql；
