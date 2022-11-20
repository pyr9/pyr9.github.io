---
title: Java获取class的五种方式
date: 2022-07-13 22:32:32
tags: 
- java基础
categories: Java基础
---

# 获取Class 类实例的五种方式

## 1.已知具体的类，直接取该类的class属性(最安全，最可靠)

```java
Class clazz01 = Person.class;
```

## 2.已知某个类的示例，调用该示例的getClass()

```java
Class clazz02 = new Person().getClass();
```

##  3.已知该类的全路径，通过Class.forName() 获得

```java
 Class class03 = Class.forName("com.getClass.pojo.Person");
```

该方式可能会抛出：ClassNotFoundException

## 4.已知该类的全路径，通过ClassLoader.loadClass() 获得

```java
ClassLoader classLoader = MySpringApplicationContext.class.getClassLoader();
final Class clazz05 = classLoader.loadClass("com.getClass.pojo.Person");
```

该方式可能会抛出：ClassNotFoundException

## 5. 基本了类型的包装类，直接调用.type

```java
Class clazz04 = Integer.TYPE;
```

​			
