---
title: java命令--jstack的使用
date: 2022-06-25 22:50:24
tags:
- java命令
categories: JVM
---

jstack用于生成java虚拟机当前时刻的线程快照。可以用来分析线程问题（如死锁），CPU突然飙升问题

# 1. 案例一 找出死锁

## 1.1 问题代码

```java
public class DeadLockTest {

   private static Object lock1 = new Object();
   private static Object lock2 = new Object();

   public static void main(String[] args) {
      new Thread(() -> {
         synchronized (lock1) {
            try {
               System.out.println("thread1 begin");
               Thread.sleep(5000);
            } catch (InterruptedException e) {
            }
            synchronized (lock2) {
               System.out.println("thread1 end");
            }
         }
      }).start();

      new Thread(() -> {
         synchronized (lock2) {
            try {
               System.out.println("thread2 begin");
               Thread.sleep(5000);
            } catch (InterruptedException e) {
            }
            synchronized (lock1) {
               System.out.println("thread2 end");
            }
         }
      }).start();

      System.out.println("main thread end");
   }
}
```

## 1.2 问题定位

### 1.2.1 方式一 jps+jstack

1. 运行`jps`，查看当前所有java进程的pid
2. `jstack pid` 查看当前进程的堆栈状态，如果有死锁会打印出来found 1 deadLock

![image-20230228231139474](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228231139474.png)

​       可以定位出是17行出了死锁问题。

### 1.2.2 方式二 jvisualvm自动检测死锁 

代码运行起来后，启动jvisualvm，在线程页面会直接有一个红色的显示：监测到死锁

![image-20230228231151238](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228231151238.png)



# 2 案例二 找出占用cpu最高的线程堆栈信息

## 2.1 问题代码

```java
public class Math {

  public static final int initData = 666;
  public static User user = new User();

  public int compute() {  //一个方法对应一块栈帧内存区域
    int a = 1;
    int b = 2;
    int c = (a + b) * 10;
    return c;
  }

  public static void main(String[] args) {
   Math math = new Math();
    while (true) {
      math.compute();
    }
  }
}
```

## 2.2 问题定位

### 2.2.1 命令top -p <pid> ，显示你的java进程的内存情况，找到当前进程的PID

![image-20230228231203454](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228231203454.png)

### 2.2.2 按H，获取每个线程的内存情况 

### 2.2.3 找到内存和cpu占用最高的线程tid

### 2.2.4 执行 `jstack PID|grep -A 10 4cd0`

比如tid为19664 ，转为十六进制得到 0x4cd0，执行 jstack 19663|grep -A 10 4cd0，得到线程堆栈信息中 4cd0 这个线程所在行的后面10行，从堆栈中可以发现导致cpu飙高的调用方法 

![image-20230228231212070](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228231212070.png)

6，查看对应的堆栈信息找出可能存在问题的代码 

