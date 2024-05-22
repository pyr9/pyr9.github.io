---
title: Jpa和mybatis
date: 2024-05-22 18:16:54
tags:
categories:
---

# 1. JPA和mybatis的区别

**基本概念**

- **MyBatis**：
  - 是一个半自动化的持久层框架，专注于 SQL 映射和管理。
  - 提供 SQL 查询、结果映射和存储过程的调用，允许开发者编写自己的 SQL。
- **JPA**：
  - 是 Java EE 和 Java SE 的标准规范，用于对象关系映射（ORM）。
  - 主要实现包括 Hibernate、spring data jpa 和 OpenJPA。
  - 提供全自动化的 ORM 功能，开发者通常不需要编写 SQL，而是依赖于 JPA 提供的抽象层。

**2. SQL 操作**

- **MyBatis**：
  - 强调手写 SQL，开发者有完全的 SQL 控制权。
  - 通过 XML 文件或注解定义 SQL 映射。
  - 更适合对 SQL 有细粒度控制需求的场景。
- **JPA**：
  - 使用 JPQL（Java Persistence Query Language）或 Criteria API 查询，类似于 SQL，但面向对象。
  - 复杂查询下可以使用QueryDsL 和写nativeSQL
  - 更侧重于通过实体类操作数据库，减少直接 SQL 的编写。
  - 对 SQL 的控制较少，但方便数据操作。
  - 更换数据库平台方便

**3. 配置和复杂度**

- **MyBatis**：
  - 配置较为简单，主要通过 XML 文件或注解。
  - 需要手动管理 SQL 和对象映射，适合复杂 SQL 或存储过程的项目。
- **JPA**：
  - 配置相对复杂，依赖于实体类、关系映射和持久化单元配置。
  - 通过注解或 XML 文件定义实体类及其关系，自动管理实体的生命周期。

**4 性能和灵活性**

- **MyBatis**：
  - 性能较好，尤其在复杂查询和批量操作方面。
  - 灵活性高，开发者可以优化每个 SQL 语句。
- **JPA**：
  - 提供缓存机制（一级和二级缓存）和延迟加载，提高性能。
  - 性能受 ORM 层次的影响较大，适合 CRUD 操作较多的应用。

**5. 学习曲线**

- **MyBatis**：
  - 学习曲线较平缓，SQL 和映射配置简单直观。
  - 适合有 SQL 基础的开发者。
- **JPA**：
  - 学习曲线较陡，需要掌握 JPA 规范和面向对象的数据库操作方法。
  - 适合需要高层次抽象和自动化管理的项目。

**6 适用场景**

- **MyBatis**：
  - 适用于需要频繁编写复杂 SQL 的项目。
  - 适合对 SQL 语句有精细控制要求的应用。
- **JPA**：
  - 适用于标准化 CRUD 操作较多的企业级应用。
  - 适合需要 ORM 特性和面向对象数据库操作的项目。

# 2. Mybatis 动态 sql 概念

摆脱根据不同条件拼接 SQL 语句的痛苦，例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。MyBatis 中用于实现动态 SQL 的元素主要有：

1. if 语句 (简单的条件判断)

2. choose (when,otherwize) ,相当于 java 语言中的 switch ,与 jstl 中的choose 很类似.
3. where (主要是用来简化 sql 语句中 where 条件判断的，能智能的处理 and or , 不必担心多余导致语法错误)
4. set (主要用于更新时，功能和 where 标签元素差不多，主要是在包含的语句前输出一个 set，然后如果包含的语句是以逗号结束的话将会把该逗号忽略，如果 set 标签最终返回的内容为空的话则可能会出错（ update set name = #{name}, where … ? )
5. trim (trim 元素的主要功能是可以在自己包含的内容前加上某些前缀，也可以在其后加上某些后缀，与之对应的属性是 prefix 和 suffix；可以把包含内容的首部某些内容覆盖，即忽略，也可以把尾部的某些内容覆盖，对应的属性是 prefixOverrides 和 suffixOverrides；正因为 trim 有这样的功能，它可以用来实现 where 和 set 的效果)
6. foreach (java中有for, 可通过for循环， 同样， mybatis中有foreach, 可通过它实现循环，循环的对象当然主要是java集合或数组。)
