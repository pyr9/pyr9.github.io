---
title: java基础
date: 2024-05-25 19:24:37
tags:
categories:
---

# 1. Java 面向对象的三大特性及理解

   默认情况下，面向对象有三大特性：继承、封装、多态。但如果考官让回答4大特性，我们就把抽象加上去。

1、继承

 继承就是从已有的类继承信息创建新类的过程，被继承的类称为父类（也叫基类、超类），继承的类叫做子类（也叫派生类）。子类可以全盘接受父类的所有属性和方法（甚至是private修饰的，也可以继承，但是不能在父类之外访问，提供共有的访问方法（比如封装set()、get()）就可以用）。

2、封装

​      封装就是把数据和操作数据的方法绑定起来，对外提供简单的接口，在java中对类中的方法的定义就是对细节的一种封装，简单地说，封装就是隐藏一切可以隐藏的内容，对外提供一个简单的接口。

3、多态

 		多态就是指不同子类型对于同一消息做出不同的反应，简单地说，就是对于同一对象，调用相同的方法，但做了不同的事情，多态具体体现在方法重载和方法重写，方法重载实现的是编译时的多态，而方法重写实现的是运行时的多态，需要注意的是当发生向上转型时，子类独有的方法不可以被调用。

4、抽象

 		抽象就是把不同的对象相同的特征总结出来提取成一个类，抽象只关注对象有哪些的属性和方法，，从不关心这些方法具体怎么实现。

# 2.什么是面向对象？什么是面向过程？

- 面向过程（Procedure Oriented 简称 PO）：把事情拆分成几个步骤（相当于拆分成一个个的方法和数据），然后按照一定的顺序执行。

- 面向对象（Object Oriented 简称 OO）：面向对象会把事物抽象成对象的概念，先抽象出对象，然后给对象赋一些属性和方法，然后让每个对象去执行自己的方法。

举例：用洗衣机洗衣服，来看一下两者的差别。

**面向过程：**

放衣服（方法）-->加洗衣粉（方法）--> 加水（方法）--> 清洗（方法）--> 甩干（方法）

**面向对象：**

1. new 出两个对象 ”人“ 和 ”洗衣机“

   - ”人“ 加入属性和方法：放衣服（方法）、加洗衣粉（方法）、加水（方法）

   - ”洗衣机“ 加入属性和方法：清洗（方法）、甩干（方法）

2. 然后执行：

​		人.放衣服（方法）-> 人.加洗衣粉（方法）-> 人.加水（方法）-> 洗衣机.清洗（方法）-> 洗衣机.甩干（方法）

# 2. 面向对象和面向过程的优缺点对比

**面向过程：**

优点：

- 效率高，因为不需要实例化对象。

缺点：

- 代码维护和扩展相对困难，特别是在大型系统中。
- 数据和行为分离，修改数据结构可能影响多个函数。

**面向对象：**

优点：

- 代码维护和扩展相对容易，通过继承和多态实现扩展。
- 封装和模块化设计使得系统更易维护和扩展。

缺点：效率比面向过程低。



# 3. Java为什么要有类

### 1. **面向对象编程的核心**

类是面向对象编程（OOP）的基本构建模块。面向对象编程是一种通过将数据和行为封装在对象中的编程范式。类提供了一种定义和创建对象的模板，使得面向对象编程成为可能。

### 2. **封装数据和行为**

类允许将数据（字段）和行为（方法）封装在一起。这种封装有助于组织代码，增强代码的可维护性和可读性，并保护数据不被外部直接访问或修改。

### 3. **实现抽象**

类允许创建抽象的数据类型，通过隐藏实现细节，只暴露必要的接口给外部使用者。这种抽象能力使得复杂系统可以分解为更简单、更可管理的部分。

### 4. **继承和代码复用**

类支持继承，可以从现有的类派生出新的类，从而实现代码的复用和扩展。子类继承父类的属性和方法，可以重用现有代码并添加新的功能。

### 5. **多态性**

类支持多态性，这使得相同接口的不同实现可以以统一的方式使用。多态性通过方法重写和接口实现实现，允许代码更加灵活和可扩展。

### 6. **模块化和组织代码**

类提供了一种自然的方式来组织和模块化代码。在大型项目中，使用类可以将相关的功能、数据和行为集中在一个模块中，使代码更易于理解、管理和维护。



# 4. java如何创建一个线程？

1. **通过继承`Thread`类**：

- 创建一个新的类继承`Thread`类。
- 重写`run`方法，在`run`方法中定义线程要执行的任务。
- 创建该类的实例并调用`start`方法启动线程。

```java
public class MyThread extends Thread {
    @Override
    public void run() {
        // 线程要执行的任务
        System.out.println("Thread is running");
    }

    public static void main(String[] args) {
        MyThread thread = new MyThread();
        thread.start();  // 启动线程
    }
}
```

2. **通过实现`Runnable`接口**：

- 创建一个实现`Runnable`接口的类。
- 实现`run`方法，在`run`方法中定义线程要执行的任务。
- 创建一个`Thread`对象，并将实现`Runnable`接口的实例作为参数传递给`Thread`构造函数。
- 调用`Thread`对象的`start`方法启动线程。

```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        // 线程要执行的任务
        System.out.println("Thread is running");
    }

    public static void main(String[] args) {
        MyRunnable myRunnable = new MyRunnable();
        Thread thread = new Thread(myRunnable);
        thread.start();  // 启动线程
    }
}
```



3. **通过实现 `Callable` 接口并使用 `FutureTask`**：

通过使用 `Callable` 和 `FutureTask`，你可以创建一个带返回结果的线程，并在需要时获取其结果。这种方式非常适合用于需要并行执行并获取结果的任务。

```java
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

public class SimpleCallableExample {

    public static void main(String[] args) {
        // 创建一个实现 Callable 接口的类
        Callable<Integer> callableTask = new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                // 模拟一些计算任务
                System.out.println("Thread is running");
                return 42;  // 返回结果
            }
        };

        // 使用 FutureTask 包装 Callable 对象
        FutureTask<Integer> futureTask = new FutureTask<>(callableTask);

        // 创建并启动线程
        Thread thread = new Thread(futureTask);
        thread.start();

        try {
            // 获取异步计算的结果
            Integer result = futureTask.get();
            System.out.println("Result from thread: " + result);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    }
}

```

3. **通过使用`ExecutorService`框架**：

- 使用`Executors`创建一个线程池。
- 提交实现`Runnable`或`Callable`接口的任务给线程池来执行。

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class MyRunnable implements Runnable {
    @Override
    public void run() {
        // 线程要执行的任务
        System.out.println("Thread is running");
    }

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(2);
        MyRunnable myRunnable = new MyRunnable();
        executorService.execute(myRunnable);  // 提交任务给线程池执行
        executorService.shutdown();  // 关闭线程池
    }
}
```

# 5. Spring如何创建一个线程

1. 使用 `@Async` 注解

`@Async` 注解用于将某个方法标记为异步执行。要使用它，你需要启用Spring的异步支持。

```java
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

@Service
public class AsyncService {
    
    @Async
    public void asyncMethod() {
        System.out.println("Async method started");
        try {
            Thread.sleep(2000);  // 模拟长时间任务
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Async method finished");
    }
}

```

2. 使用 `TaskExecutor`

`TaskExecutor` 是Spring提供的一组接口，用于抽象并发编程的细节。你可以使用 `ThreadPoolTaskExecutor` 来创建一个线程池。

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.Executor;

@Configuration
public class TaskExecutorConfig {

    @Bean(name = "taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(25);
        executor.setThreadNamePrefix("AsyncThread-");
        executor.initialize();
        return executor;
    }
}

```

在服务类中注入 `TaskExecutor` 并使用它执行任务：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.task.TaskExecutor;
import org.springframework.stereotype.Service;

@Service
public class TaskExecutorService {

    @Autowired
    private TaskExecutor taskExecutor;

    public void executeTask() {
        taskExecutor.execute(() -> {
            System.out.println("Task executed by TaskExecutor");
            try {
                Thread.sleep(2000);  // 模拟长时间任务
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("Task completed");
        });
    }
}

```
