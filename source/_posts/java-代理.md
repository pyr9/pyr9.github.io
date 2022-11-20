---
title: java 代理
date: 2022-08-16 11:11:18
tags: 面试题
categories: Java基础 
---

# 代理

 代理是一种设计模式

- 定义：通过代理对象访问目标对象。
- 好处：可以在不修改目标对象的基础上，扩展目标对象的功能

 代理类可以分为静态代理和动态代理。

## **1.静态代理**和动态代理

### 静态代理

1. 定义： 通过声明一个明确的代理类来访问源对象。需要和目标对象一起实现相同的接口或者继承相同的父类。

2. 特点：在编译器确定代理对象，在程序运行前，代理类的.class文件就已经存在了。

3. 优点：可以做到不修改目标对象的前提下，对目标对象的方法进行扩展。

4. 缺点：

   - 代理类需要和目标类实现相同的接口，同时要实现相同的方法，这样就出现了很多重复代码。

   - 如果接口增加一个方法，代理类和目标类都需要实现这个方法，增加了代码维护的复杂度。

     

### 动态代理

#### JDK动态代理

1. 定义：使用JDK官方的Proxy类创建代理对象。

2. 特点：在程序运行过程中，动态生成代理类，代理类需要实现**InvocationHandler**，重写invoke方法。

3. 优点：

   - 接口中声明的所有方法被迁移到代理类中的invoke方法进行实现，这样，如果接口方法很多的话，不需要为每一个方法进行中转，可以灵活处理。

     

#### CGLIB动态代理

1. 使用：第三方CGLib的Enhancer类创建代理对象, 设置Superclass为目标类， 使用的是方法拦截器 **MethodInterceptor**，重写intercept方法。

2. 特点：Cglib是通过生成子类来实现的，代理对象既可以赋值给实现类，又可以赋值给接口。

3. 优点：Cglib速度比jdk动态代理更快，性能更好。

   

## 静态代理程序示例

step1:定义委托类

```java
public interface Email {
    public void send();
    public void receive();
}
public class FlashEmail implements Email {

	@Override
	public void send() {
		// TODO Auto-generated method stub
            System.out.println("邮件发送中。。。。。");
	}

	@Override
	public void receive() {
		// TODO Auto-generated method stub
       System.out.println("邮件接受中..........");
	}

}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

step2:定义代理类，与委托类有同样的接口

 不改变原接口实现类的情况下，就可以对接口的功能进行需求变更

```java
public class EmailProxy implements Email {

	Email email;
     //传入委托类初始化代理类
	public EmailProxy(Email email)
	{
		this.email=email;
	}
	@Override
	public void send() {
    System.out.println("发送邮件前准备。。。");
    email.send();
    System.out.println("发送后。。。。。。");
	}

	@Override
	public void revieve() {
		
		System.out.println("接受邮件前准备。。。"); 
		email.receive();
		System.out.println("接受邮件成功....");
	}

}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

step3:测试代码：

```java
public class Main {

	public static void main(String[] args) {
		//原始对象
	   Email email=new FlashEmail();
	   //接口的代理类
	   EmailProxy ep=new EmailProxy(email);
	   ep.send();
	   System.out.println("-----------------------------------------");
	   ep.receive();
	}
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

测试结果：

发送邮件前准备。。。
 邮件发送中。。。。。
 发送后。。。。。。
 \-----------------------------------------
 接受邮件前准备。。。
 邮件接受中..........
 接受邮件成功....
 

### 2 JDK动态代理程序示例

Step1: 定义接口

```java
public interface Email {
    public void send();
    public void revieve();
}
```

step1:定义委托类

```java
public class FlashEmail implements Email {

	@Override
	public void send() {
		// TODO Auto-generated method stub
            System.out.println("邮件发送中。。。。。");
	}

	@Override
	public void receive() {
		// TODO Auto-generated method stub
       System.out.println("邮件接受中..........");
	}

}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

step2:在使用动态代理时，我们需要定义一个位于代理类与委托类之间的中介类，也叫动态代理类，以完成代理要完成的具体操作，这个类被要求实现**InvocationHandler**接口：

```java
package com.my.test.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class EmailProxy implements InvocationHandler {

	Object obj;//委托对象
	
	public EmailProxy(Object obj) {
		super();
		this.obj = obj;
	}

	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		
		if("send".equals(method.getName())) {
			 System.out.println("发送邮件前准备。。。");
			 method.invoke(obj);//从委托对象中，调用该对象的指定方法，这里相当于send方法
		     System.out.println("发送后。。。。。。");
		}
		if("receive".equals(method.getName())) {
			System.out.println("接受邮件前准备。。。"); 
			method.invoke(obj);
			System.out.println("接受邮件成功....");
		}

		return null;
	}

}
```



step4:测试

```java
public class Main {
  public static void main(String[] args) {
    Email email=new FlashEmail();
    Email emailProxy=(Email) Proxy.newProxyInstance(
      email.getClass().getClassLoader(),
      email.getClass().getInterfaces(),
      new EmailProxy(email) );
    emailProxy.send();
    System.out.println("---------------------------------------------------");
    emailProxy.receive();
  }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

测试结果：

发送邮件前准备。。。
 邮件发送中。。。。。
 发送后。。。。。。
 \-----------------------------------------
 接受邮件前准备。。。
 邮件接受中..........
 接受邮件成功....

### CGLIB动态代理程序示例

Step1: 引入cglib依赖

```java
<dependency>
  <groupId>cglib</groupId>
  <artifactId>cglib</artifactId>
  <version>3.2.5</version>
  </dependency>
```

Step2: 定义委托类

```java
public class FlashEmail {
  public void send() {
    System.out.println("邮件发送中。。。。。");
  }

  public void receive() {
    System.out.println("邮件接受中..........");
  }
}
```

Step3: 创建一个拦截器，实现接口net.sf.cglib.proxy.MethodInterceptor，用于方法的拦截回调。

```java
public class LogInterceptor implements MethodInterceptor {
  @Override
  public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
    Object result = null;
    if("send".equals(method.getName())) {
      System.out.println("发送邮件前准备。。。");
      result = methodProxy.invokeSuper(proxy,args);
      System.out.println("发送后。。。。。。");
    }
    if("receive".equals(method.getName())) {
      System.out.println("接受邮件前准备。。。");
      result = methodProxy.invokeSuper(proxy,args);
      System.out.println("接受邮件成功....");
    }
    return result;
  }
}
```

Step4: 测试

```java
package proxy;

import net.sf.cglib.proxy.Enhancer;

public class Test {
  public static void main(String[] args) {
    // 创建Enhancer对象，类似于JDK动态代理的Proxy类
    Enhancer enhancer = new Enhancer();
    enhancer.setSuperclass(FlashEmail.class);
    enhancer.setCallback(new LogInterceptor());
    // create方法正式创建代理类
    FlashEmail emailProxy = (FlashEmail)enhancer.create();
    emailProxy.send();
    emailProxy.receive();
  }
}
```

测试结果：

发送邮件前准备。。。
 邮件发送中。。。。。
 发送后。。。。。。
 \-----------------------------------------
 接受邮件前准备。。。
 邮件接受中..........
 接受邮件成功....
