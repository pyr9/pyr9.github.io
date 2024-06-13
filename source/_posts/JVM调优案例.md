---
title: JVM调优案例
date: 2022-09-14 17:29:19
tags:
- 性能调优
categories: JVM调优
---

# 1. 案例一 ——测试环境服务异常挂掉

## 1. 问题

测试环境服务异常挂掉

## 2. 排查步骤

查看操作系统日志，发现发生了OOM，操作系统的OOM KIiller 机制 将Java进程杀掉了（使用命令` egrep -i 'Out of memory' /var/log/messages`）

> [root@sith-mom-test-node deploy]# egrep -i 'Out of memory' /var/log/messages
> May 28 10:19:07 sith-mom-test-node kernel: Out of memory: Kill process 12420 (java) score 161 or sacrifice child
> May 28 10:19:07 sith-mom-test-node kernel: Out of memory: Kill process 12607 (java) score 161 or sacrifice child
> May 28 10:19:07 sith-mom-test-node kernel: Out of memory: Kill process 12721 (java) score 161 or sacrifice child
> May 28 17:07:24 sith-mom-test-node kernel: Out of memory: Kill process 10434 (java) score 92 or sacrifice child
> May 28 17:34:42 sith-mom-test-node kernel: Out of memory: Kill process 9270 (java) score 75 or sacrifice child
> May 28 17:34:42 sith-mom-test-node kernel: Out of memory: Kill process 9281 (java) score 75 or sacrifice child
> May 28 17:50:18 sith-mom-test-node kernel: Out of memory: Kill process 24060 (java) score 64 or sacrifice child

重新启动项目，使用`df -h`发现测试环境内存空闲也就20G，使用`jstat -gc 67825 3000 1000` 发现元空间占了10G以上，还在一直增长。（测试环境内存62G），因为jdk.8之后元空间使用的内存是本地内存，默认是-1， 即不限制，所以怀疑到了元空间

## 3. 修复

在启动jar包时，加上了`-XX:MaxMetaspaceSize=20G`, 指定最大元空间大小, 问题修复，后续又感觉20G占用太大，又尝试降低10G， 5G， 目前设置的是1000M，5天Full GC 了200次，平均2小时3次， 一次5.8秒，还在持续调整（期待1小时一次Full GC, 一次1s）。



# 3.案例二 ——天气预报系统

天气预报系统，有一个展示所有城市天气预报的页面，由于城市比较多，需要分页去展示，于是先设置了一次查询5000条，由于每一条城市都需要再去给他设置对应的天气信息，所以搞了一个foreach每一条记录都需要new City()，然后再去set天气信息

初始参数是：

堆的最大和最小容量都设置了1536M, 年轻代是512M，使用的ParNew+CMS垃圾收集器，元空间256M

```
-Xms1536M -Xmx1536M -Xmn512M -Xss256K -XX:SurvivorRatio=6 -XX:MetaspaceSize=256M -XX:MaxMetaspaceSize=512M  -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly
```

## 3.2 观察GC整体情况

- step1 : 运行` jps`看下, 查看当前java进程PID

  ![image-20230228222821739](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228222821739.png)

- step2: 运行 `jstat -gc 2593 3000 10000`，每隔3秒，打印下GC压力整体情况。观察可发现，程序突然开始频繁Young Gc 和Full GC。

![image-20230228222903395](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228222903395.png)

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

![image-20230228222923471](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228222923471.png)

这种情况我们可以借助jmap或者jvisualvm命令大概看下是什么对象

![image-20230228222946533](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228222946533.png)

查到了有大量User对象产生，这个可能是问题所在，但不确定，还必须找到对应的代码确认，如何去找对应的代码了？ 

1、代码里全文搜索生成User对象的地方(适合只有少数几处地方的情况) 

2、如果生成User对象的地方太多，无法定位具体代码，我们可以同时分析下占用cpu较高的线程，一般有大量对象不断产生，对应的方法 代码肯定会被频繁调用，占用的cpu必然较高 

![image-20230228222956571](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228222956571.png)

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

![image-20230228223006680](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228223006680.png)



# 

# 3. 案例三—— 订单系统

有一个订单系统，需要支持每秒支持下300个订单。假定每个订单对象是1kb，那么每秒将会有300kb对象生成。

下单可能还涉及创建其他对象创建，比如库存，优惠卷，积分，以及这个系统可能还支持订单的查询，我们放大20倍，那么就是300 * 20  = 6000kb，约60MB，也就是这个系统每秒会产生60M的垃圾。

假设机器是4核8G，那么一般可能会给我们虚拟机分配个四五G的内存，就会给堆分配个3G的内存，那么方法区分配256M，单个栈内存分配1M。

```java
1 ‐Xms3072M ‐Xmx3072M ‐Xss1M ‐XX:MetaspaceSize=256M ‐XX:MaxMetaspaceSize=256M ‐XX:SurvivorRatio=8
```

那么这么分配有什么问题呢？如果是一个并发量不大的系统，基本上也不会有什么问题，因为本身也没有多少对象在堆里生成。但是如果是我们上面说的亿级流量系统，就不能简简单单这么设置了，我们来分析一下：

堆分配的大小是3G，按照1:2分配年轻代和老年代的话，可以算出年轻代是1G，老年代是2G，年轻代再按照8:1:1细分eden和survivor，eden是800M，单个survivor是100M。如果每秒产生60M对象，优先在eden分配，也就是差不多13秒，eden就会被占满，发生minor GC，这是90%的对象已经被回收了，还存在一些正在使用的对象，假定是60M对象还不会被回收。

那么这60M对象会从Eden区移到S0区域，由于存在动态年龄判断，就是一批对象的总大小大于这块Survivor区域内存大小的，那么此时**大于等于**这批对象年龄最大值的对象，就可以直接进入老年代了。这样的话，每13秒就会有60M的对象被移入老年代。

那么大概五六分钟老年代的2G就会被占满，那么老年代一满就要发生full gc

![image-20230228224246219](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228224246219.png)



[(129条消息) JVM的STW(stop the world)机制及调优案例_JavaRecord的博客-CSDN博客_stw机制](https://blog.csdn.net/weixin_44704538/article/details/108222022)



来分析一下：我们这个系统其实没有很多会长久存在的对象，也就是老不死的对象，我们放在老年代的那些60M的对象，在一两秒后其实都会变为垃圾对象，在下一次full gc时都会被回收掉，那么我们这种业务场景，完全没必要给老年代设置2G的内存，根本用不到。

我们完全可以把年轻代设置的大一些，我们现在来对jvm参数进行一些更改。

```java
1 ‐Xms3072M ‐Xmx3072M ‐Xmn2048M ‐Xss1M ‐XX:MetaspaceSize=256M ‐XX:MaxMetaspaceSize=256M ‐XX:SurvivorRatio=8
```



此时需要25秒把伊甸园区放满，放满minor gc后有60M对象不被回收，要移到S0区，这时60M<200M/2，是可以移入S0区域的，下一次伊甸园区再放满做minor gc的时候，这时这60M对象所对应的订单已经生成了，已经变成了垃圾对象，是可以直接被回收的，所以没有什么对象是需要被移入老年代的。

那么这么一设置的话，这个系统是不是正常情况下基本不会再发生full gc了呢？就算发生，也是很久才会一次了。

![image-20230228224257622](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228224257622.png)
