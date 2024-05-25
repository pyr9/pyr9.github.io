---
title: spring常见面试题
date: 2024-05-24 15:10:23
tags:
categories:
---

# 1. Spring 注入过程

1. 启动 Spring 容器
2. 读取配置文件或注解扫描（如 `@ComponentScan`）加载 Bean 的定义。
3. Spring 容器会根据 Bean 的定义实例化对象。这个过程称为实例化（Instantiation）。
4. Spring 容器会注入依赖对象到 Bean 中。依赖注入可以通过构造器、Setter 方法或字段注入来实现。

依赖注入完成后，可以通过 Spring 上下文获取 Bean 实例。



# 2. Spring两大特性

Spring 是一个开源框架，Spring可以做很多事情，它为企业级开发提供这些功能的底层都依赖于它的两个核心特性：IOC和AOP。

1. IOC：IOC控制反转也叫依赖注入，IOC 利用 java 反射机制，所谓控制反转是指，本来被调用者的实例是有调用者来创建的，这样的缺点是耦合性太强，IOC 则是统一交给 spring 容器来管理创建，将对象交给容器管理，你只需要在 spring 配置文件中配置相应的 bean，以及设置相关的属性，让spring 容器来生成类的实例对象以及管理对象。就比如：原来我的service层要调用dao层，service就要创建dao,但spring发现你的service层要依赖dao层后，就可以给你注入。

2. AOP：为 Aspect Oriented Programming 的缩写，意为：面向切面编程，可以通过预编译方式和运行期动态代理实现在不修改源代码的情况下给程序动态统一添加功能的一种技术。
   （1）比如说现在有一个类 iA，里面有增删改的方法，现在要在每个目标方法执行前后进行开启事务和提交事务。传统的思想是纵向继承：要想实现事务的功能的话，不得不采用 子类继承父类的方法，通过重写父类的目标方法，在此基础上加上事务，开启事务/提交事务，这样的话，代码重复率很高，每个方法都要写开启事务/提交事务，代码重复率太高。
   （2）AOP 采取横向抽取机制：就是说，鉴于纵向继承出现重复的代码，把重复的代码提取出来放到一个类 B 中，然后 spring 生成一个代理类，他和目标类也就是类 A 实现相同的接口，拥有相同的方法，那么代理类就可以把类 B 和目标类都拿来，进行组合，对外访问的话，就是访问的代理类。
   （3）就达到和纵向继承一样的效果。他在 spring 中是用反射去做，我们要做的就是编写目标类 A 和切面类类 B。然后告诉 spring，让 Spring 生成代理就行了，代码少了很多，但是功能一个不少。

# 3. spring优点
（1）方便解耦，简化开发（高内聚，低耦合）
Spring就是一个容器，可以将所有对象创建和依赖关系维护，都交给spring管理，spring工厂用于生成bean
（2）AOP编程的支持
可以通过预编译方式和运行期动态代理实现在不修改源代码的情况下给程序动态统一添加功能的一种技术
（3）声明式事务的支持
只需要通过配置就可以完成对事务的管理，无需手动编程
（4）方便程序的测试
spring对Junit4支持，可以通过注解方便的测试spring程序
（5）方便集成各种优秀框架
spring内部提供了对各种优秀框架的支持，如mybatis等

# 4. 在 Spring 中，有几种依赖注入方式？

通常，依赖注入可以通过三种方式完成
即：构造函数注入、setter 注入、接口注入，在 Spring Framework 中，仅使用构造函数和 setter 注入。

```
<!-- 1、构造方法注入 --> 
    <bean id="teacher1" class="com.pojo.Teacher">
       <constructor-arg name="name" value="老刘"></constructor-arg>
        <constructor-arg name="age" value="36"></constructor-arg>
    </bean>

<!--  setter方法注入 -->  
    <bean id="stu" class="com.pojo.Student">
       <property name="name" value="lisi44"></property>
       <property name="age" value="99"></property>
   </bean>

```

# 5 在 Spring 中，有几种配置 Bean 的方式？

三种，分别为：基于XML的配置、基于注解的配置、基于Java的配置

1.基于XML的配置:

```
<bean id="stu" class="com.pojo.Student">
       <property name="name" value="lisi"></property>
       <property name="age" value="22"></property>
   </bean>
```

2.基于注解的配置

```java
@Autowired
Student stu;
```

3.基于java 显式配置

```
@Bean
	public Clazz createClazz() {
		
		Clazz clazz=new Clazz();
		clazz.setName("自动化专业");
		return clazz;
	}
```
