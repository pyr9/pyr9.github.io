---
title: Spring Data jpa
date: 2022-10-16 16:03:46
tags: JPA
categories: JPA
---

# 1.Java访问持久层的两种方式

- 以Sql为核心，封装一定程度的JDBC操作，比如mybatis
- 以Java实体类为核心，将实体类和数据表之间建立连接，也就是我们说的ORM框架，比如：Hibernate,SpringData jpa

# 2. 什么是JPA

-  JPA统一了Java应用程序访问ORM框架的规范

**JPA为我们提供了以下规范：**

1. ORM映射元数据：JPA支持XML和注解两种元数据的形式，元数据描述对象和表之间的映射关系，框架据此将实体对象持久化到数据库表中
2.  JPA 的API：用来操作实体对象，执行CRUD操作，框架在后台替我们完成所有的事情，**开发人员不用再写SQL**了
3.  JPQL查询语言：通过**面向对象而非面向数据库的查询语言查询数据**，避免程序的SQL语句紧密耦合。

# 3  JPA和Hibernate的关系

• JPA是一个规范，而不是框架

• Hibernate是JPA的一种实现，是一个框架

# 4 **Spring Data**是什么？

Spring Data是Spring 社区的一个子项目，主要用于简化数据（关系型&非关系型）访问，其主要目标是使得数据库的访问变得方便快捷。•它提供很多模板操作

-  Spring Data Elasticsearch

-  Spring Data MongoDB

- Spring Data Redis

- Spring Data Solr

# 5 **Spring Data JPA**是什么？

- Spring Data JPA是在实现了JPA规范的基础上封装的一套 JPA 应用框架。
- Spring Data JPA旨在通过将统一ORM框架的访问持久层的操作，来提高开发人的效率。
- 虽然ORM框架都实现了JPA规范，但是在不同的ORM框架之间切换仍然需要编写不同的代码，而使用Spring Data JPA能够方便大家在不同的ORM框架之间进行切换而不需要更改代码。

## 5.1 **Spring Data JPA给我们提供的主要的类和接口**

- Repository 接口：

  - Repository

  - CrudRepository

  - JpaRepository

- Repository 实现类：

  - SimpleJpaRepository

  - QueryDslJpaRepository

# 6 **Spring Data JPA和Hibernate的关系**

- Hibernate其实是JPA的一种实现，而Spring Data JPA是一个JPA数据访问抽象。
- pring Data JPA不是一个实现或JPA提供的程序，它只是一个抽象层，主要用于减少为各种持久层存储实现数据访问层所需的样板代码量。但是它还是需要JPA提供实现程序。
- Spring Data JPA是一种JPA的抽象层，底层依赖Hibernate

# 7 mybatis 和hibernate的对比？

（1）hibernate是全自动，而mybatis是半自动。

hibernate完全可以通过对象关系模型实现对数据库的操作，拥有完整的JavaBean对象与数据库的映射结构来自动生成sql。而mybatis仅有基本的字段映射，对象数据以及对象实际关系仍然需要通过手写sql来实现和管理。

（2）hibernate数据库移植性远大于mybatis

hibernate通过它强大的映射结构和hql语言，大大降低了对象与数据库（Oracle、MySQL等）的耦合性，而mybatis由于需要手写sql，因此与数据库的耦合性直接取决于程序员写sql的方法，如果sql不具通用性而用了很多某数据库特性的sql语句的话，移植性也会随之降低很多，成本很高。

（3）hibernate拥有完整的日志系统，mybatis则欠缺一些

hibernate日志系统非常健全，涉及广泛，包括：sql记录、关系异常、优化警告、缓存提示、脏数据警告等；而mybatis则除了基本记录功能外，功能薄弱很多。

（4）sql直接优化上，mybatis要比hibernate方便很多

由于mybatis的sql都是写在xml里，因此优化sql比hibernate方便很多。而hibernate的sql很多都是自动生成的，无法直接维护sql；
