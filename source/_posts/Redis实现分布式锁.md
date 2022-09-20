---
title: Redis实现分布式锁
date: 2022-09-20 15:03:35
tags:
- 分布式锁
categories: redis
---

## 1 **基于Redis实现分布式锁**

效率最高，加锁速度最快，因为Redis几乎都是纯内存操作

适用于并发量很大、性能要求很高而可靠性问题可以通过其 他方案去弥补的场景。 

## 1.1 设计原理

利用Redis的`SETNX key value`这个命令，只有当key不存在时才会执行成功，如果key已经存在则命令执行失败。

## 1.2.2 实现注意事项

- 需要设置一个超时时间，因为有可能宕机或者被运维重启了，无法释放锁
- 如果业务执行时间> 过期时间，就需要**锁续命**：搞一个定时任务，设一个间隔时间，小于失效时间，过一段时间去监控业务是否执行完了，执行没结束，也就是锁还没释放，我就再把锁的过期时间重新设置成初始值

## redis分布式锁

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h6b11433hoj21kg0u0adf.jpg)

- 多个线程，只有一个线程会执行成功
- 必须使用try catch 在finally中释放锁，否则有异常，锁便没法释放，其他线程进来就会一直执行失败
- 分布式锁需要设置一个超时时间，因为有可能宕机或者被运维重启了，需要保证原子性，写成一个命令

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h6b1621v7yj21ec0goq5r.jpg)

存在问题：现在有两个线程A，B，加锁时长设置的是10秒

- 线程A先去加锁，执行完所有的逻辑需要 15秒，他加锁成功后，正在执行下面的逻辑，第10秒的时候，锁被释放了
- 线程B加锁成功，执行完所有的逻辑需要 8，执行完逻辑后，删除这把锁
- 线程A逻辑执行完了，去释放锁，但锁已经被线程B释放掉了

解决方式，给每一个线程分配一个UUID

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h6b1hwkaarj21jw0scn1d.jpg)

存在问题：

获取client id 和删除redis必须是原子性操作，因为可能存在线程A执行到删除锁之前刚好到失效时间了，然后又卡顿了，这时候线程B就又可以获得锁，释放锁，导致线程A又没法释放锁

解决方式：

锁续命：搞一个定时任务，设一个间隔时间，小于失效时间，过一段时间去监控业务是否执行完了，执行没结束，也就是锁还没释放，我就再把锁的过期时间重新设置成初始值



redison

使用：

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h6b1ygqebkj217s0ccwfs.jpg)

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h6b1z3x7w3j20eq034gll.jpg)

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h6b1wzvpx3j21dn0u0tck.jpg)

原理：

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h6b22djvmej21260o8whq.jpg)



# 
