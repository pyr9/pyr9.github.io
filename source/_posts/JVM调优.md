---
title: JVM调优
date: 2022-09-17 16:57:45
tags:
- 性能调优
- JVM
categories: JVM
---

# 1 调优原则

- **多数导致GC问题的Java应⽤，都不是因为我们参数设置错误，⽽是代码问题**；

- 实际使用中，分析GC优化代码，比优化JVM参数效果要好，JVM 参数的默认（推荐）值都是经过 JVM 团队的反复测试和前人的充分验证得出的比较合理的值，因此通常来说是比较靠谱和通用的，一般不会出大问题。
- 如果满足下面的指标，则一般不需要进行GC：
  - Minor GC执行时间不到50ms，不低于10秒一次；
  - 
    Full GC执行时间不到1s，不低于一小时1次；

# 2. 检查Full GC 是否频繁

可以通过`jstat -gc pid interval count`，每隔interval秒，打印count次，pid进程下GC压力整体情况。观察是否出现频繁Full GC

<img src="https://panyuro.oss-cn-beijing.aliyuncs.com/image-20240612121617021.png" alt="image-20240612121617021" style="zoom: 100%;" />

参数说明：

- **S0C** 和 **S1C**：幸存区 0 和幸存区 1 的容量分别为 14336 KB 和 10752 KB。

- **S0U** 和 **S1U**：幸存区 0 没有使用，幸存区 1 使用了 10724.4 KB。

- **EC** 和 **EU**：伊甸园区的容量为 215040 KB，使用了 20511.6 KB。
- **OC** 和 **OU**：老年代的容量为 139776 KB，使用了 18056.9 KB。
- **MC** 和 **MU**：元空间的容量为 45184 KB，使用了 42025.9 KB。
- **CCSC** 和 **CCSU**：压缩类空间的容量为 6272 KB，使用了 5667.6 KB。
- **YGC** 和 **YGCT**：年轻代垃圾回收次数为 5 次，垃圾回收总时间为 0.068 秒。
- **FGC** 和 **FGCT**：完全垃圾回收次数为 2 次，垃圾回收总时间为 0.054 秒。
- **CGC** 和 **CGCT**：并发垃圾回收次数和时间未显示（可能是因为没有启用或发生）。
- **GCT**：垃圾回收总时间为 0.122 秒。

# 3  JVM参数

可以考虑[JVM内存模型](https://pyr9.github.io/JVM%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B/)：

设置格式：

```java
java ‐Xms2048M ‐Xmx2048M ‐Xmn1024M ‐Xss512K ‐XX:MetaspaceSize=256M ‐XX:MaxMetaspaceSize=256M ‐jar microservice‐eureka‐server.jar 
```

**堆**：

- `-Xms`：设置初始堆大小。
- `-Xmx`：设置最大堆大小。

**年轻代：**

- `-Xmn<size>`：设置年轻代内存大小。（老年代的大小等于堆内存大小减去年轻代的大小）

**Eden区与Survivor区的比例：**

- `-XX:SurvivorRatio=<ratio>`：设置Eden区与Survivor区的比例。

**年轻代与老年代**：

- `-XX:NewRatio=<ratio>`：设置年轻代与老年代的比例。

**永久代/元空间：**

- `-XX:MaxPermSize=<size>`：设置永久代的最大大小（仅适用于Java 7及之前版本）。

- `-XX:MetaspaceSize=<size>`：设置元空间的初始大小。

- `-XX:MaxMetaspaceSize=<size>`：设置元空间的最大大小（适用于Java 8及之后版本）

**栈**：

- `-Xss<size>`：设置每个线程的堆栈大小。

**垃圾收集器**

- `-XX:+UseSerialGC`：使用串行垃圾收集器。
- `-XX:+UseParallelGC`：使用并行垃圾收集器（Parallel GC）。
- `-XX:+UseConcMarkSweepGC`：使用并发标记清除垃圾收集器（CMS）。
- `-XX:+UseG1GC`：使用G1垃圾收集器。
- `-XX:+UseZGC`：使用ZGC垃圾收集器。
- `-XX:+UseShenandoahGC`：使用Shenandoah垃圾收集器。

**垃圾收集调优**

- `-XX:MaxGCPauseMillis=<ms>`：设置最大GC暂停时间。
- `-XX:G1HeapRegionSize=<size>`：设置G1垃圾收集器的堆区域大小。
- `-XX:InitiatingHeapOccupancyPercent=<percent>`：设置触发混合GC的堆占用百分比（用于G1）。
- `-XX:ParallelGCThreads=<n>`：设置用于并行垃圾收集的线程数。

# 4. JVM调优案例

[案例](https://pyr9.github.io/JVM%E8%B0%83%E4%BC%98%E6%A1%88%E4%BE%8B/)
