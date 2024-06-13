---
title: java命令--jmap工具
date: 2022-09-13 18:29:57
tags:
- java命令
- 性能调优
categories: JVM
---

# **Jmap** 

命令jmap是一个多功能的命令。它可以生成 java 程序的 dump 文件， 也可以查看堆内对象示例的统计信息、查看 ClassLoader 的信息以及 finalizer 队列。

**系统中内存飙升了，怎么办？**

使用jmap看看是否有大量类生成

![image-20230228231003015](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228231003015.png)

### - histo

查看堆内存信息，实例个数以及占用内存大小 

- 命令： ```  ~ jmap -histo 91656 > ./log.txt ```
- 结果分析
  - num：序号 
  - instances：实例数量 
  - bytes：占用空间大小 
  - class name：类名称   [C is a char[]，[S is a short[]，[I is a int[]，[B is a byte[]，[[I is a int[][] 

![](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228231003015.png)

### ‐dump

生成Java虚拟机的堆转储快照dump文件

- 命令 `jmap -dump:live,format=b,file=heap.bin <pid>`

- 运行结果

  ```java
  ➜  ~ jmap -dump:live,format=b,file=eureka.hprof 3548
  Dumping heap to /Users/panyurou/eureka.hprof ...
  ```

- 也可以设置内存溢出自动导出dump文件(内存很大的时候，可能会导不出来) 

  - -XX:+HeapDumpOnOutOfMemoryError 
  -  -XX:HeapDumpPath=./ （路径）

- **示例代码：**

  ```java
  public class OOMTest {
    // JVM设置 :‐Xms10M ‐Xmx10M ‐XX:+PrintGCDetails
    // ‐XX:+HeapDumpOnOutOfMemoryError ‐XX:HeapDumpPath=/Users/panyurou/jvm.hprof
    public static void main(String[] args) {
      List<Object> list = new ArrayList<>();
      int i = 0;
      int j = 0;
      while (true) {
        list.add(new User(i++));
        new User(j--);
      }
    }
  }
  ```

  运行配置：

  ![image-20230228231024765](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228231024765.png)

  运行结果：

  ![image-20230228231032270](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228231032270.png)

- 用jvisualvm命令工具导入该dump文件分析

  ![image-20230228231039259](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228231039259.png)
