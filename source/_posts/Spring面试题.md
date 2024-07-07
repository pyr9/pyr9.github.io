---
title: Spring知识点整理
date: 2022-08-16 09:51:49
tags:
- 整理
categories: Java基础
---

# 1. Spring是什么？有什么优点

- Spring是一个轻量级的java框架，Spring的核心是IOC和AOP。主要的优点包括：
  - 方便解耦，简化开发。通过Spring提供的IOC容器，我们可以将对象的依赖关系交给Spring进行控制，避免硬编码造成的耦合性高。简单来说，之前，如果我们controller 需要一个service，则需要new service，现在是Spring发现你的controller 需要service，就给你自动注入。
  - AOP编程的支持。Spring提供面向切面编程，方便的实现对程序进行拦截，运行监控等。主要通过BeanPostProcessor 的postProcessBeforeInitialization，postProcessAfterInitialization， + 动态代理实现。（有接口就用JDK的动态代理，否则CJLIB）
  - 声明式事务的支持。只需要配置就可以实现对事务的管理，而无需手动编程。
  - 对Junit支持，方便程序测试。
    - 集成了各种优秀框架，如mybatis, quartz。

# 2 在 Spring 中，Bean 是如何生成的？/ bean 的生命周期

- Spring扫描class 文件，得到BeanDefinition
- 根据BeanDefinition去生成bean
  - 实例化阶段：根据构造函数来完成实例化 （**未属性注入以及初始化的对象** 这里简称为 **原始对象**）
  - 属性注入阶段：对 Bean 的属性进行依赖注入 （这里就是**发生循环依赖问题的环节**）
  - 如果 Bean 的某个方法有AOP操作，则需要根据原始对象生成**代理对象**。（beanPostProcessor）
- 最后把代理对象放入单例池（一级缓存singletonObjects）中，下次获取对象就直接从单例池中获取。

# 3. Spring的循环依赖

循环依赖就是在创建bean的过程中（实例化 Bean 后会进行 属性注入（依赖注入）），A对象依赖了B对象，B对象依赖了A对象，导致A，B两个对象都创建不出来。

>  ABean 创建–>依赖了 B 属性–>触发 BBean 创建—>B 依赖了 A 属性—>需要 ABean（但 ABean 还在创建过程中）

而这种情况只会在将Bean交给Spring管理的时候才会出现，因为上面的这些属性注入操作都是Spring去做的，如果只是我们自己在Java中创建对象可以不去注入属性，让成员属性为NULL也可以正常执行的，这样也就不会出现循环依赖的问题了。

在 Spring 中，通过**三级缓存**机制帮开发者解决了部分循环依赖的问题。
- 一级缓存为：**singletonObjects**: 缓存已经经历了完整生命周期的 bean 对象。
- 二级缓存为：**earlySingletonObjects**: 缓存的是早期的 bean 对象。表示 Bean 的生命周期还没走完就把这个 Bean 放入了 earlySingletonObjects。
- 三级缓存为：**singletonFactories**: 缓存的是 ObjectFactory，对象工厂，用来创建早期 bean 对象的工厂。实际上是一个 Lambda 表达式， 执行lambda表达式，得到就是对应的代理对象。

简单点说，Spring解决循环依赖的思路就是：当A的bean需要B的bean的时候，提前将A的bean放在缓存中（实际是将A的ObjectFactory放到三级缓存），然后再去创建B的bean，但是B的bean也需要A的bean，那么这个时候就去缓存中拿A的bean，B的bean创建完毕后，再回来继续创建A的bean，最终完成循环依赖的解决。

Spring 利用 三级缓存 巧妙地将出现 循环依赖 时的 AOP 操作 提前到了 属性注入 之前（通过第三级缓存实现的），解决了循环依赖问题。


# 4 Spring前后置环绕异常通知

- before: 方法执行前执行，如果通知抛出异常，则方法无法执行。
- after: 方法执行后执行，不论方法中是否抛出异常。主要用来清理现场。
- around：环绕通知，方法执行前后分别执行，必须手动调用目标方法。
- afterReturning: 方法执行后执行，可以获得方法的返回值，方法中抛出异常则无法执行。
- afterThrowing: 方法抛出异常后执行。 

# 5 Spring事务传播行为（多个事务存在是怎么处理的？）

- equired(需要)： 默认值。如果当前存在事务则加入这个事务，如果当前没有事务就新创建一个。

- supports(支持)：如果当前存在事务，就加入这个事务，否则就以非事务的方式运行。

- Mandatory(必须，强制)：如果当前存在事务，就加入这个事务，如果当前没有事务，就抛出异常。

- Requires_new: 如果当前存在事务，就将该事务挂起。创建一个新的事务来运行。

- Not_supported: 如果当前存在事务，就将该事务挂起。以非事务的方式运行。

- never:  如果当前存在事务，就抛出异常，以非事务的方式运行。

- nested(嵌套)：  如果当前存在事务，就创建一个事务当作当前事务的嵌套事务来运行，如果当前没有事务，就创建一个事务。

  (加入*3+ 挂起* * 2+ 异常 * 1 + 嵌套事务 *1 s)

# 6. @Resource和@Autowired的区别？

1. 来源不同

@Autowired 是 Spring 定义的注解，而 @Resource 是 JDK 定义的注解，

2. 依赖查找顺序不同

- @Autowired 是先根据类型（byType）查找，如果存在多个 Bean 再根据名称（byName）进行查找，，可以通过 `@Qualifier` 注解指定要注入的 bean 名称。

- @Resource 是先根据名称查找，如果（根据名称）查找不到，再根据类型进行查找

3. 依赖注入的用法支持不同

- @Autowired 既支持构造方法注入，又支持属性注入和 Setter 注入
- @Resource 只支持属性注入和 Setter 注入；
