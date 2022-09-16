---
title: JVM调优案例(一)——日均百万级订单交易系统如何设置JVM参数
date: 2022-09-14 17:29:19
tags:
- 性能调优
categories: JVM
---

# 场景分析

有一个订单系统，需要支持每秒支持下300个订单。假定每个订单对象是1kb，那么每秒将会有300kb对象生成。

下单可能还涉及创建其他对象创建，比如库存，优惠卷，积分，以及这个系统可能还支持订单的查询，我们放大20倍，那么就是300 * 20  = 6000kb，约60MB，也就是这个系统每秒会产生60M的垃圾。

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h669kt0unej20qi0yq763.jpg" style="zoom:67%;" />

假设机器是4核8G，那么一般可能会给我们虚拟机分配个四五G的内存，就会给堆分配个3G的内存，那么方法区分配256M，单个栈内存分配1M。

```java
1 ‐Xms3072M ‐Xmx3072M ‐Xss1M ‐XX:MetaspaceSize=256M ‐XX:MaxMetaspaceSize=256M ‐XX:SurvivorRatio=8
```

那么这么分配有什么问题呢？如果是一个并发量不大的系统，基本上也不会有什么问题，因为本身也没有多少对象在堆里生成。但是如果是我们上面说的亿级流量系统，就不能简简单单这么设置了，我们来分析一下：

堆分配的大小是3G，按照1:2分配年轻代和老年代的话，可以算出年轻代是1G，老年代是2G，年轻代再按照8:1:1细分eden和survivor，eden是800M，单个survivor是100M。如果每秒产生60M对象，优先在eden分配，也就是差不多13秒，eden就会被占满，发生minor GC，这是90%的对象已经被回收了，还存在一些正在使用的对象，假定是60M对象还不会被回收。

那么这60M对象会从Eden区移到S0区域，由于存在动态年龄判断，就是一批对象的总大小大于这块Survivor区域内存大小的，那么此时**大于等于**这批对象年龄最大值的对象，就可以直接进入老年代了。这样的话，每13秒就会有60M的对象被移入老年代。

那么大概五六分钟老年代的2G就会被占满，那么老年代一满就要发生full gc

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h66e35154hj20t80bmab3.jpg" style="zoom:50%;" />





[(129条消息) JVM的STW(stop the world)机制及调优案例_JavaRecord的博客-CSDN博客_stw机制](https://blog.csdn.net/weixin_44704538/article/details/108222022)







来分析一下：我们这个系统其实没有很多会长久存在的对象，也就是老不死的对象，我们放在老年代的那些60M的对象，在一两秒后其实都会变为垃圾对象，在下一次full gc时都会被回收掉，那么我们这种业务场景，完全没必要给老年代设置2G的内存，根本用不到。

我们完全可以把年轻代设置的大一些，我们现在来对jvm参数进行一些更改。

```java
1 ‐Xms3072M ‐Xmx3072M ‐Xmn2048M ‐Xss1M ‐XX:MetaspaceSize=256M ‐XX:MaxMetaspaceSize=256M ‐XX:SurvivorRatio=8
```



此时需要25秒把伊甸园区放满，放满minor gc后有60M对象不被回收，要移到S0区，这时60M<200M/2，是可以移入S0区域的，下一次伊甸园区再放满做minor gc的时候，这时这60M对象所对应的订单已经生成了，已经变成了垃圾对象，是可以直接被回收的，所以没有什么对象是需要被移入老年代的。

那么这么一设置的话，这个系统是不是正常情况下基本不会再发生full gc了呢？就算发生，也是很久才会一次了。

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h66e2iqgokj21dw0i8gok.jpg" style="zoom:50%;" />
