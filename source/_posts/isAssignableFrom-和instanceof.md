---
title: isAssignableFrom()
date: 2022-07-17 16:24:24
tags: java基础
categories: Java基础
---

# isAssignableFrom()

A.isAssignableFrom(B)方法可用来判断B是否可以由A转换（赋值得来），即

1.当A是类时，表示：类A是否为B类的父类

```java
package com.my.test.isAggsignableFrom;

class Parent{
}

class Son1 extends Parent{
}

class Brother {
}
public class Test01 {
  public static void main(String[] args) {
    System.out.println(Parent.class.isAssignableFrom(Son1.class));   => true
    System.out.println(Parent.class.isAssignableFrom(Brother.class));  => false
  }
}
```



2.当A是接口时，表示：接口A是否被类B实现

```java
package com.my.test.isAggsignableFrom;
interface A{
}

class A1 implements A{
}

class B1{
}

public class Test02 {
  public static void main(String[] args) {
    System.out.println(A.class.isAssignableFrom(A1.class));  => true
    System.out.println(A.class.isAssignableFrom(B1.class));  => false
  } 
}

```

