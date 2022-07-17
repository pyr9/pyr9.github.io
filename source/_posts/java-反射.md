---
title: java 反射
date: 2022-07-17 16:43:34
tags: java基础
categories: java
---

# 反射是什么？

- 反射就是指在运行状态中，对于任何一个类，能够知道这个类的所有属性和方法。

- 对于任何一个对象，都能调用它的任意属性和方法，并且可以改变他的属性。

- 反射是java被市委准动态语言的关键，反射机制允许程序在执行期间借助反射Api来获取任何类的内部信息，并能够直接操作任意对象的内部属性及方法。
- 加载完类后，在堆内存的方法区就产生了一个Class类型的对象，对应这个类。这个对象就包含了这个类的完整结构信息，我们可以通过这个对象看到类的结构。这个对象就像一面镜子，通过这面镜子我们可以看到类的结构，所以我们称之为：反射。

# 反射可以做什么

反射允许我们在程序运行时，取得任何一个已知class的内部信息，包括其属性，方法等，并可以在运行时改变他们呢。

获取class对象的五种方式：[Java获取class的五种方式 - 楼上有只喵 (panyurou.github.io)](https://panyurou.github.io/2022/07/13/Java获取class的四种方式/)

# 反射的具体实现

查阅 API 可以看到 Class 有很多方法：

- getName()：获得类的完整名字。
- getFields()：获得类的public类型的属性。

  - getDeclaredFields()：获得类的所有属性。包括private 声明的和继承类
  - getMethods()：获得类的public类型的方法。
  - getDeclaredMethods()：获得类的所有方法。包括private 声明的和继承类
  - getMethod(String name, Class[] parameterTypes)：获得类的特定方法，name参数指定方法的名字，parameterTypes 参数指定方法的参数类型。
  - getConstructors()：获得类的public类型的构造方法。
  - getConstructor(Class[] parameterTypes)：获得类的特定构造方法，parameterTypes 参数指定构造方法的参数类
  - newInstance()：通过类的不带参数的构造方法创建这个类的一个对象

**代码测试：**

```java
package com.my.test.reflection;

class Father{
  public String fatherId;
  private String fatherName;

  public String getFatherHobbies(){
    return "eat";
  }

  private String getFatherWork(){
   return "computer";
  }
}
public class Son extends Father{
  public String sonId;
  private String name;
  private Integer age;

  public String getSonHobbies(){
    return "study";
  }

  private String getSonWork(){
    return "student";
  }
}
```

```java
package com.my.test.reflection;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class Test {
  public static void main(String[] args) throws NoSuchFieldException, NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
    final Class<Son> clazz = Son.class;
    System.out.println("获得类的完整名字-------------------");
    System.out.println(clazz.getName());

    System.out.println("获得类的public类型的属性-------------------");
    for (Field field : clazz.getFields()) {
      // 由于JDK的安全检查耗时较多.所以通过setAccessible(true)的方式关闭安全检查就可以达到提升反射速度的目的
      field.setAccessible(true);
      System.out.println(field.getName());
    }

    System.out.println("获得类的所有属性。包括private的---------------");
    for (Field field : clazz.getDeclaredFields()) {
      System.out.println(field.getName());
    }


    System.out.println("获得类的public类型的方法,包括Object的-------------------");
    for (Method method : clazz.getMethods()) {
      System.out.println(method.getName());
    }

    System.out.println("获得类的所有方法。-------------------");
    for (Method method : clazz.getDeclaredMethods()) {
      System.out.println(method.getName());
    }

    System.out.println("获得指定的public属性。-------------------");
    Field sonId = clazz.getField("sonId");
    System.out.println(sonId);

    System.out.println("获得指定的private属性。-------------------");
    Field age = clazz.getDeclaredField("age");
    age.setAccessible(true);
    System.out.println(age);

    System.out.println("创建这个类的一个对象-------------------");
    final Son son = clazz.getDeclaredConstructor().newInstance();
    System.out.println(son);

    System.out.println("将son对象的age属性赋值为19------------");
    age.set(son, 19);
    System.out.println(age.get(son));

    System.out.println("获取构造方法-------------");
    final Constructor<?>[] constructors = clazz.getConstructors();
    for (Constructor<?> constructor : constructors) {
      System.out.println(constructor.toString());
    }
  }
}
```

Console:

```
获得类的完整名字-------------------
com.my.test.reflection.Son
获得类的public类型的属性-------------------
sonId
fatherId
获得类的所有属性。包括private的---------------
sonId
name
age
获得类的public类型的方法,包括Object的-------------------
getSonHobbies
getFatherHobbies
wait
wait
wait
equals
toString
hashCode
getClass
notify
notifyAll
获得类的所有方法。-------------------
getSonWork
getSonHobbies
getFatherHobbies
获得指定的public属性。-------------------
public java.lang.String com.my.test.reflection.Son.sonId
获得指定的private属性。-------------------
private java.lang.Integer com.my.test.reflection.Son.age
创建这个类的一个对象-------------------
com.my.test.reflection.Son@1cd072a9
将son对象的age属性赋值为19------------
19
获取构造方法-------------
public com.my.test.reflection.Son()
```

