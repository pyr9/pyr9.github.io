---
title: JVM调优
date: 2022-09-17 16:57:45
tags:
- 性能调优
- JVM
categories: JVM调优
---

# 1 调优原则

- **多数导致GC问题的Java应⽤，都不是因为我们参数设置错误，⽽是代码问题**；

- 实际使用中，分析GC优化代码，比优化JVM参数效果要好，JVM 参数的默认（推荐）值都是经过 JVM 团队的反复测试和前人的充分验证得出的比较合理的值，因此通常来说是比较靠谱和通用的，一般不会出大问题。
- 如果满足下面的指标，则一般不需要进行GC：
  - Minor GC执行时间不到50ms，不低于10秒一次；
  - 
    Full GC执行时间不到1s，不低于一小时1次；

# 2 调优步骤

## 2.1 内存分析

- 方式： 检查是否有OutOfMemory 等内存异常， 检查哪些对象在系统中数量最大，避免频繁生成新对象，类似天气预报系统里有一个分页展示，每次批量去操作5000个对象，会频繁触发Minor GC 和 Full GC，后面调成500个，基本上就没有Full GC了。
- 工具：jmap ‐dump，生成Java虚拟机的堆转储快照dump文件，用jvisualvm命令工具导入该dump文件分析，最多的类是哪些
- 参考地址：[java命令--jmap工具 - 楼上有只喵 (panyurou.github.io)](https://panyurou.github.io/java命令--jmap工具/)

## 2.2 死锁检查

- 方式一：jps+jstack

  - 运行`jps`，查看当前所有java进程的pid
  - `jstack pid` 查看当前进程的堆栈状态，如果有死锁会打印出来found 1 deadLock

- 方式二：jvisualvm

   代码运行起来后，启动jvisualvm，在线程页面会直接有一个红色的显示：监测到死锁

- 参考地址：[java命令--jstack的使用 - 楼上有只喵 (panyurou.github.io)](https://panyurou.github.io/java命令-jstack-工具/)

## 2.3 CPU分析，哪些方法占用的大量CPU时间

- 方式：
  - 命令top -p ，显示你的java进程的内存情况，找到当前进程的PID
  - top -Hp，获取每个线程的内存情况，获取CPU比较高的线程id，并专成16进制
  - 执行 `jstack PID|grep -A 10 4cd0`得到线程堆栈信息，找出可能存在问题的代码，比如大量计算，或者循环new 对象

## 2.4  垃圾回收器选择

- **JDK 1.8默认使用 Parallel**，**JDK 1.9默之后认使用 G1**
- 100M以下用Serial，简单+高效
- 4G以下可以用parallel，并发收集 + 高吞吐量。（标记整理，STW）
- 4-8G可以用ParNew+CMS，并发收集 + 低停顿(注重用户体验）
- 8G以上可以用G1，避免长时间的停顿+可预期的GC停顿周期 + 高吞吐量(筛选回收时需要STW)
- 几百G以上用ZGC，支持TB量级的堆+ 最大GC停顿时间不超10ms

## 2.5 Full GC 次数频繁

可以通过`jstat -gc pid interval count`，每隔interval秒，打印count次，pid进程下GC压力整体情况。观察是否出现频繁Full GC

### 2.5.1 调用System.gc()方法的调用

- 此方法的调用是建议JVM进行Full GC,虽然只是建议而非一定,但很多情况下它会触发 Full GC,从而增加Full GC的频率
- **调优建议**：能不使用此方法就别使用，让虚拟机自己去管理它的内存

### 2.5.2 方法区空间不足 

- 方法区存放的为一些class的信息、常量、静态变量等数据，当系统中要加载的类、反射的类和调用的方法较多时，方法区可能会被占满，从而执行Full GC。
- 元空间的初始阈值设置太小，垃圾收集器调整元空间的大小需要Full GC。比如初始值是21M，full gc后，释放了20M，只剩1M，那么垃圾收集器就会将该值调成2M；相反，如果释放了1M，还剩20M，下次就会调整到40M，这个过程会多次触发full GC.
- 一般建议在JVM参数中将MetaspaceSize和MaxMetaspaceSize设置成一样的值，并设置得比初始值要大。
- **调优建议：**对于8G物理内存的机器来说，一般我会将这两个值都设置为256M。常见的问题：项目jar包只有1G，但是启动了好久，就可能是因为由于元空间设置的太小，一直在触发full GC。

### 2.5.3 老年代空间不足

**对象转入老年代的场景有：**

- 大对象直接进入老年代：为了避免为大对象GC时，多次在survivor上来回复制而降低效率。
  - **调优建议：**
    - 尽量避免写大对象，
    - 结合你自己系统看下有没有什么大对象 生成，预估下大对象的大小，设置XX:PretenureSizeThreshold的值，一般来说设置为1M就差不多了，很少有超过1M的大对象，这些对象一般就是你系统初始 化分配的缓存对象，比如大的缓存List，Map之类的对象。
- 长期存活的对象进入老年代：默认是经历了15次minor GC ，CMS是6次。
  - **调优建议：**
    - 调整Eden和survivor的比例，默认时8:1:1，避免Eden区域过小，导致频繁minor GC，增大系统消耗
    - 如果我们可以确定大多数对象都是很快会被回收的，可以将该值调小，比如大多数对象都是3次左右就可以被回收，我们可以设置成5，如果超过了5，就说明剩下的应该是一些类似java bean 这种，会长期存活的对象，我们就没必要，让他再待在survivor占用空间了，提前让他们进入老年代。
- **对象动态年龄判断**：survivor区域里，对象年龄从小到大排序，当累加到某个年龄时，占用 survivor区域 的50%，就会把这个年龄大的对象都晋升到老年代。 如果系统中朝生夕死的对象比较多，
  - **调优建议：**
    - 如果系统中朝生夕死的对象比较多，可以考虑将年轻代参数调大一些，这样对应survivor区域就会对应变大，减少对象因为触发了这个机制，进入到老年代，从而触发Full GC。一般情况，新生代默认是堆大小的1/3
    - 避免Survivor设置过小，导致对象从 eden 直接到达老年代，从而触发Full GC

- 老年代空间分配担保机制。 

  在发生 Minor GC 之前，虚拟机先检查老年代最大可用的连续空间是否大于新生代所有对象总空间。

  - 大于 -> minor GC
  - 小于 ->查看是否开启了空间担保机制
    - 没开启就直接触发full GC，
    - 开启了，就会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试着进行一次 Minor GC；如果小于，会触发full gc。

# 3.调优案例

## 3.1 场景

天气预报系统，有一个展示所有城市天气预报的页面，由于城市比较多，需要分页去展示，于是先设置了一次查询5000条，由于每一条城市都需要再去给他设置对应的天气信息，所以搞了一个foreach每一条记录都需要new City()，然后再去set天气信息

初始参数是：

堆的最大和最小容量都设置了1536M, 年轻代是512M，使用的ParNew+CMS垃圾收集器，元空间256M

```
-Xms1536M -Xmx1536M -Xmn512M -Xss256K -XX:SurvivorRatio=6 -XX:MetaspaceSize=256M -XX:MaxMetaspaceSize=512M  -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly
```

## 3.2 观察GC整体情况

- step1 : 运行` jps`看下, 查看当前java进程PID

  <img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h68t6kw94aj20bs05m0sx.jpg" style="zoom:100%;" />

- step2: 运行 `jstat -gc 2593 3000 10000`，每隔3秒，打印下GC压力整体情况。观察可发现，程序突然开始频繁Young Gc 和Full GC。

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h6avlmvphuj21pc0lmdql.jpg)

## 3.2 思考频繁Full Gc 触发条件

- 调用System.gc()方法的调用。   **-> 不可能，系统中没这样调用**


- 云空间，方法区空间不足  **-> 不可能，观察MU并没有很大变化，说明元空间没有一直扩容，所以空间是够的**


- 老年代空间不足，对象转入老年代的场景有：
  - 大对象直接进入老年代：为了避免为大对象GC时，多次在survivor上来回复制而降低效率。   **-> 不可能，系统中没很多大对象**
  - 长期存活的对象进入老年代：默认是经历了15次minor GC ，CMS是6次 **-> 可能，代码在频繁生成对象，且占用周期比较长。有可能是代码有问题，其实我们可以频繁生成对象，但是应该让他朝生夕死。**
  - **对象动态年龄判断**：survivor区域里，对象年龄从小到大排序，当累加到某个年龄时，占用 survivor区域 的50%，就会把这个年龄大的对象都晋升到老年代。  **-> 可能，因为现在是频繁生成了对象，可以考虑年轻代调大一些。**

- 老年代空间分配担保机制。 **-> 可能性不大，因为老年代挺大的**

## 3.3 调优步骤

如果是对于对象动态年龄判断机制导致的full gc较为频繁可以先试着优化下JVM参数，把年轻代适当调大点： -Xmn1024M

```java
-Xms1536M -Xmx1536M -Xmn1024M -Xss256K -XX:SurvivorRatio=6 -XX:MetaspaceSize=256M -XX:MaxMetaspaceSize=512M  -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly
```

我们发现young gc和full gc依然很频繁了，而且看到有大量的对象频繁的被挪动到老年代

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h6avmoyunkj21nf0u0tnc.jpg)这种情况我们可以借助jmap或者jvisualvm命令大概看下是什么对象

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h6avoot12aj21k80u0q9y.jpg)

查到了有大量User对象产生，这个可能是问题所在，但不确定，还必须找到对应的代码确认，如何去找对应的代码了？ 

1、代码里全文搜索生成User对象的地方(适合只有少数几处地方的情况) 

2、如果生成User对象的地方太多，无法定位具体代码，我们可以同时分析下占用cpu较高的线程，一般有大量对象不断产生，对应的方法 代码肯定会被频繁调用，占用的cpu必然较高 

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h6avprtd0tj21h20u012w.jpg)

可以用上面讲过的jstack或jvisualvm来定位cpu使用较高的代码，最终定位到的代码如下： 

```java

@RestController
@RequestMapping("/report")
public class WeatherReportController {
    /**
   * 模拟批量查询城市天气预报场景
   */
    private ArrayList<City> queryCities() {
    ArrayList<City> cities = new ArrayList<>();
    for (int i = 0; i < 5000; i++) {
      cities.add(new City(String.valueOf(i),"深圳"));
    }
    return cities;
  }
}
```

```java

public class City {
  private String id;

  private String name;

  private String code;

  byte[] info = new byte[1024*100];
}
```

同时，java的代码也是需要优化的，一次查询出500M的对象出来，明显不合适，要根据之前说的各种原则尽量优化到合适的值，尽量消除这种朝生夕死的对象导致的full gc

改成批量查询500后，Full GC 基本不再发生

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h6avzmzupaj21ru0mqgwc.jpg)



# 4 **内存泄露到底是怎么回事**

一般电商架构可能会使用多级缓存架构，就是redis加上JVM级缓存，大多数同学可能为了图方便对于JVM级缓存就简单使用一个hashmap，于是不断往里面放缓存数据，但是很少考虑这个map的容量问题，结果这个缓存map越来越大，一直占用着老年代的很多空间，时间长了就会导致full gc非常频繁，这就是一种内存泄漏，对于一些老旧数据没有及时清理导致一直占用着宝贵的内存资源，时间长了除了导致full gc，还有可能导致OOM。 

这种情况完全可以考虑采用一些成熟的JVM级缓存框架来解决，比如ehcache等自带一些LRU数据淘汰算法的框架来作为JVM级的缓存。 

