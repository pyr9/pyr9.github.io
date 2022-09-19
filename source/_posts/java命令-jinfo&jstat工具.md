---
title: java命令--jinfo&jstat
date: 2022-06-19 21:36:07
tags:
- java命令
categories: JVM调优
---

# **Jstat** 

jstat命令可以查看堆内存各部分的使用量，以及加载类的数量。

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h67521v5p3j21060lmadr.jpg)



## 垃圾回收统计

**jstat -gc pid 最常用**，可以评估程序内存使用及GC压力整体情况 

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h678gi8orqj21no03274w.jpg)

- S0C：第一个幸存区的大小，单位KB

- S1C：第二个幸存区的大小

- S0U：第一个幸存区的使用大小

- S1U：第二个幸存区的使用大小

- EC：伊甸园区的大小

- EU：伊甸园区的使用大小

- OC：老年代大小

- OU：老年代使用大小

- MC：方法区大小(元空间)

- MU：方法区使用大小

- CCSC:压缩类空间大小

- CCSU:压缩类空间使用大小

- YGC：年轻代垃圾回收次数，指的是程序从运行起来到现在的次数

- YGCT：年轻代垃圾回收消耗时间，单位s

- FGC：老年代垃圾回收次数 

- FGCT：老年代垃圾回收消耗时间，单位s

- GCT：垃圾回收消耗总时间，单位s

  

 **jstat -gc pid 1000 10 (每隔1秒执行1次命令，共执行10次)**

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h678iroyusj21o404u404.jpg)



## 堆内存统计

命令：`jstat -gccapacity 3548`

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h678m9ifvgj21tk02uaau.jpg)

- NGCMN：新生代最小容量
- NGCMX：新生代最大容量
- NGC：当前新生代容量
- S0C：第一个幸存区大小
- S1C：第二个幸存区的大小
- EC：伊甸园区的大小
- OGCMN：老年代最小容量
- OGCMX：老年代最大容量
- OGC：当前老年代大小
- OC:当前老年代大小
- MCMN:最小元数据容量
- MCMX：最大元数据容量
- MC：当前元数据空间大小
- CCSMN：最小压缩类空间大小
- CCSMX：最大压缩类空间大小
- CCSC：当前压缩类空间大小
- YGC：年轻代gc次数
- FGC：老年代GC次数



## 新生代垃圾回收统计

 命令：`jstat -gcnew 3548`

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h678n8ozf5j20us02qq36.jpg)

- S0C：第一个幸存区的大小
- S1C：第二个幸存区的大小
- S0U：第一个幸存区的使用大小
- S1U：第二个幸存区的使用大小
- TT:对象在新生代存活的次数
- MTT:对象在新生代存活的最大次数
- DSS:期望的幸存区大小
- EC：伊甸园区的大小
- EU：伊甸园区的使用大小
- YGC：年轻代垃圾回收次数
- YGCT：年轻代垃圾回收消耗时间



## 新生代内存统计

`jstat -gcnewcapacity 3548`

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h678oy5h2zj217i02y3yy.jpg)

- NGCMN：新生代最小容量
- NGCMX：新生代最大容量
- NGC：当前新生代容量
- S0CMX：最大幸存1区大小
- S0C：当前幸存1区大小
- S1CMX：最大幸存2区大小
- S1C：当前幸存2区大小
- ECMX：最大伊甸园区大小
- EC：当前伊甸园区大小
- YGC：年轻代垃圾回收次数
- FGC：老年代回收次数



## **JVM运行情况预估**

用 jstat gc -pid 命令可以计算出如下一些关键数据，有了这些数据就可以采用之前介绍过的优化思路，先给自己的系统设置一些初始性的JVM参数，比如堆内存大小，年轻代大小，Eden和Survivor的比例，老年代的大小，大对象的阈值，大龄对象进入老年代的阈值等。

**年轻代对象增长的速率**

可以执行命令 jstat -gc pid 1000 10 (每隔1秒执行1次命令，共执行10次)，通过观察EU(eden区的使用)来估算每秒eden大概新增多少对象，如果系统负载不高，可以把频率1秒换成1分钟，甚至10分钟来观察整体情况。注意，一般系统可能有高峰期和日常期，所以需要在不同的时间分别估算不同情况下对象增长速率。

**Young GC的触发频率和每次耗时**

知道年轻代对象增长速率我们就能推根据eden区的大小推算出Young GC大概多久触发一次，Young GC的平均耗时可以通过 YGCT/YGC 公式算出，根据结果我们大概就能知道**系统大概多久会因为Young GC的执行而卡顿多久。**

**每次Young GC后有多少对象存活和进入老年代**

这个因为之前已经大概知道Young GC的频率，假设是每5分钟一次，那么可以执行命令 jstat -gc pid 300000 10 ，观察每次结果eden，survivor和老年代使用的变化情况，在每次gc后eden区使用一般会大幅减少，survivor和老年代都有可能增长，这些增长的对象就是每次Young GC后存活的对象，同时还可以看出每次Young GC后进去老年代大概多少对象，从而可以推算出**老年代对象增长速率。**

**Full GC的触发频率和每次耗时**

知道了老年代对象的增长速率就可以推算出Full GC的触发频率了，Full GC的每次耗时可以用公式 FGCT/FGC 计算得出。

**优化思路**其实简单来说就是尽量让每次Young GC后的存活对象小于Survivor区域的50%，都留存在年轻代里。尽量别让对象进入老年代。尽量减少Full GC的频率，避免频繁Full GC对JVM性能的影响。

# **Jinfo**

查看正在运行的Java应用程序的扩展参数 以及jvm的参数 

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h6751095p4j20v20c675s.jpg)

##  -flags

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h674zwtq82j219605u0uk.jpg)

