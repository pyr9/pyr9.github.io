---
title: java命令--jstack 工具
date: 2022-09-15 09:50:24
tags:
- java命令
categories: 性能调优
---

# **jstack**

jstack用于生成java虚拟机当前时刻的线程快照。可以用来分析线程问题（如死锁），CPU突然飙升问题

## 案例一： jstack找出死锁

- 代码示例

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

- 运行结果：

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h671lb0pnvj21d50u0q9l.jpg)

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h671lmfphjj212w0rs438.jpg)

​       可以定位出是17行出了死锁问题。

- 还可以用jvisualvm自动检测死锁 

  ![](https://tva1.sinaimg.cn/large/e6c9d24ely1h671owo3ajj21y20u012b.jpg)



## 案例二：jstack找出占用cpu最高的线程堆栈信息

## 1.top

在服务器上，我们可以通过top命令查看各个进程的cpu使用情况，它默认是按cpu使用率由高到低排序的

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h672961ju6j215k0b6ju6.jpg)

### 2. jstack pid

通过top命令定位到cpu占用率较高的线程之后，接着使用jstack pid命令来查看当前java进程的堆栈状态，

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h672bftvhsj214a0lo795.jpg)

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h672cg23khj20su0ng0up.jpg" style="zoom:50%;" />

