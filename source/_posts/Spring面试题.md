---
title: Spring知识点整理
date: 2022-08-16 09:51:49
tags:
- 整理
categories: Java基础
---

#### Spring是什么？有什么优点

- Spring是一个轻量级的java框架，Spring的核心是IOC和AOP。主要的优点包括：
  - 方便解耦，简化开发。通过Spring提供的IOC容器，我们可以将对象的依赖关系交给Spring进行控制，避免硬编码造成的耦合性高。简单来说，之前，如果我们controller 需要一个service，则需要new service，现在是Spring发现你的controller 需要service，就给你自动注入。
  - AOP编程的支持。Spring提供面向切面编程，方便的实现对程序进行拦截，运行监控等。主要通过BeanPostProcessor 的postProcessBeforeInitialization，postProcessAfterInitialization， + 动态代理实现。（有接口就用JDK的动态代理，否则CJLIB）
  - 声明式事务的支持。只需要配置就可以实现对事务的管理，而无需手动编程。
  - 对Junit支持，方便程序测试。
    - 集成了各种优秀框架，如mybatis, quartz。

### 在 Spring 中，Bean 是如何生成的？/ bean 的生命周期

- Spring扫描class 文件，得到BeanDefinition
- 根据BeanDefinition去生成bean
  - 根据class推断构造方法
  - 根据构造方法，利用反射，生成对象（原始对象）。
  - 填充对象中依赖的属性（依赖注入）
  - 如果对象被AOP了，就需要根据原始对象生成一个代理对象。（beanPostProcessor）
- 最终把生成的对象放进单例池（singleton objects），下次获取对象就直接从单例池中获取。



### Spring的循环依赖

- 循环依赖就在创建bean的过程中，A对象依赖了B对象，B对象依赖了A对象，导致A，B两个对象都创建不出来。

  >  ABean 创建–>依赖了 B 属性–>触发 BBean 创建—>B 依赖了 A 属性—>需要 ABean（但 ABean 还在创建过程中）

- 在 Spring 中，通过**三级缓存**机制帮开发者解决了部分循环依赖的问题。
  - 一级缓存为：**singletonObjects**: 缓存已经经历了完整生命周期的 bean 对象。
  - 二级缓存为：**earlySingletonObjects**: 缓存的是早期的 bean 对象。表示 Bean 的生命周期还没走完就把这个 Bean 放入了 earlySingletonObjects。
  - 三级缓存为：**singletonFactories**: 缓存的是 ObjectFactory，对象工厂，用来创建早期 bean 对象的工厂。实际上是一个 Lambda 表达式， 执行lambda表达式，得到就是对应的代理对象。



### Spring前后置环绕异常通知

- before: 方法执行前执行，如果通知抛出异常，则方法无法执行。
- after: 方法执行后执行，不论方法中是否抛出异常。主要用来清理现场。
- around：环绕通知，方法执行前后分别执行，必须手动调用目标方法。
- afterReturning: 方法执行后执行，可以获得方法的返回值，方法中抛出异常则无法执行。
- afterThrowing: 方法抛出异常后执行。 



### Spring事务传播行为（多个事务存在是怎么处理的？）

- equired(需要)： 默认值。如果当前存在事务则加入这个事务，如果当前没有事务就新创建一个。

- supports(支持)：如果当前存在事务，就加入这个事务，否则就以非事务的方式运行。

- Mandatory(必须，强制)：如果当前存在事务，就加入这个事务，如果当前没有事务，就抛出异常。

- Requires_new: 如果当前存在事务，就将该事务挂起。创建一个新的事务来运行。

- Not_supported: 如果当前存在事务，就将该事务挂起。以非事务的方式运行。

- never:  如果当前存在事务，就抛出异常，以非事务的方式运行。

- nested(嵌套)：  如果当前存在事务，就创建一个事务当作当前事务的嵌套事务来运行，如果当前没有事务，就创建一个事务。

  (加入*3+ 挂起* * 2+ 异常 * 1 + 嵌套事务 *1 s)
