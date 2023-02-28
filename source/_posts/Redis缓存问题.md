---
title: Redis缓存问题
date: 2021-12-29 22:45:20
tags: 
- 缓存
categories: Redis
---

# 1 **缓存穿透**

## 1.1 定义

缓存穿透是指查询一个根本不存在的数据， 缓存层和存储层都不会命中， 导致不存在的数据每次请求都要到存储层去查询， 失去了缓存保护后端存储的意义。 

## 1.2 产生原因

-  自身业务代码或者数据出现问题。 
- 一些恶意攻击、 爬虫等造成大量空命中。 

##  1.3 解决方案

### 1.3.1 缓存空对象 

```java
String get(String key) {
    // 从缓存中获取数据
    String cacheValue = cache.get(key);
    // 缓存为空
    if (StringUtils.isBlank(cacheValue)) {
        // 从存储中获取
        String storageValue = storage.get(key);
        cache.set(key, storageValue);
        // 如果存储数据为空， 需要设置一个过期时间(300秒)
        if (storageValue == null) {
            cache.expire(key, 60 * 5);
        }
        return storageValue;
    } else {
        // 缓存非空
        return cacheValue;
    }
}
```

### 1.3.2 布隆过滤器 

在访问缓存前，先用布隆过滤器先做一次过滤。对于不存在的数据布隆过滤器一般都能够过滤掉，不让请求再往后端发送。

布隆过滤器底层就是实现了一个大型的二进制数组，我们需要在使用前将所有的数据放入布隆过滤器，并且在增加数据时也要往布隆过滤器里放，布隆过滤器可以根据多个hash算法对key进行hash+数组长度取模的方式，给每一条数据计算出多个位置。

所以在访问数据是否存在时，他也会把这几个位置算出来，如果这几个位置有一个为0，那么就说明这个数据一定不存在。因为存在hash碰撞，所以如果所有位都为1，不能说明这个key一定存在。

![image-20230228225351935](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228225351935.png)

```java
package com.redisson;

import org.redisson.Redisson;
import org.redisson.api.RBloomFilter;
import org.redisson.api.RedissonClient;
import org.redisson.config.Config;

public class RedissonBloomFilter {

    public static void main(String[] args) {
        Config config = new Config();
        config.useSingleServer().setAddress("redis://localhost:6379");
        //构造Redisson
        RedissonClient redisson = Redisson.create(config);

        RBloomFilter<String> bloomFilter = redisson.getBloomFilter("nameList");
        //初始化布隆过滤器：预计元素为100000000L,误差率为3%,根据这两个参数会计算出底层的bit数组大小
        bloomFilter.tryInit(100000000L,0.03);
      
        //把所有数据存入布隆过滤器
        void init(){
            for (String key: keys) {
                bloomFilter.put(key);
            }
        }

        String get(String key) {
            // 从布隆过滤器这一级缓存判断下key是否存在
            Boolean exist = bloomFilter.contains(key);
            if(!exist){
                return "";
            }
            // 从缓存中获取数据
            String cacheValue = cache.get(key);
            // 缓存为空
            if (StringUtils.isBlank(cacheValue)) {
                // 从存储中获取
                String storageValue = storage.get(key);
                cache.set(key, storageValue);
                // 如果存储数据为空， 需要设置一个过期时间(300秒)
                if (storageValue == null) {
                    cache.expire(key, 60 * 5);
                }
                return storageValue;
            } else {
                // 缓存非空
                return cacheValue;
            }
        }
    }
}
```



# 2 **缓存失效(击穿)** 

## 2.1 定义

由于大批量缓存在同一时间失效可能导致大量请求同时穿透缓存直达数据库，可能会造成数据库瞬间压力过大 甚至挂掉。

## 2.2 产生原因

大批量缓存在同一时间失效

## 2.3 解决方案

### 2.3.1 将这一批数据的缓存过期时间设置为一个时间段内的不同时间。 

```java
String get(String key) {
    // 从缓存中获取数据
    String cacheValue = cache.get(key);
    // 缓存为空
    if (StringUtils.isBlank(cacheValue)) {
        // 从存储中获取
        String storageValue = storage.get(key);
        cache.set(key, storageValue);
        //设置一个过期时间(300到600之间的一个随机数)
        int expireTime = new Random().nextInt(300)  + 300;
        if (storageValue == null) {
            cache.expire(key, expireTime);
        }
        return storageValue;
    } else {
        // 缓存非空
        return cacheValue;
    }
}
```



# 3 **缓存雪崩** 

## 3.1 定义

缓存雪崩指的是缓存层支撑不住或宕掉后， 流量打向后端存储层 ，存储层的调用量会暴增， 造成存储层也会级联宕机的情况。

## 3.2 产生原因

- redis集群挂了
- 请求数量大于redis可承受的最大数量

## 3.3 解决方案

### 3.3.1 保证缓存层服务高可用性

比如使用Redis Sentinel或Redis Cluster。 

### 3.3.2 后端限流熔断并降级。

比如使用Sentinel或Hystrix限流降级组件。 

- 比如服务降级，我们可以针对不同的数据采取不同的处理方式。
  - 当业务应用访问的是非核心数据（例如电商商 品属性，用户信息等）时，暂时停止从缓存中查询这些数据，而是直接返回预定义的默认降级信息、空值或是错误提示信息；
  - 当业务应用访问的是核心数据（例如电商商品库存）时，仍然允许查询缓存，如果缓存缺失， 也可以继续通过数据库读取。 



# 4 **热点缓存key重建优化** 

## 4.1定义

当前key是一个热点key（例如一个热门的娱乐新闻），并发量非常大。 重建缓存不能在短时间完成， 可能是一个复杂计算， 例如复杂的SQL、 多次IO、 多个依赖等。 

## 4.2 产生原因

在缓存失效的瞬间， 有大量线程来重建缓存， 造成后端负载加大， 甚至可能会让应用崩溃。

## 4.3 解决方案

### 4.3.1 互斥锁

利用互斥锁来解决，此方法只允许一个线程重建缓存， 其他线程等待重建缓存的线程执行完， 重新从缓存获取数据即可。

```java
String get(String key) {
    // 从Redis中获取数据
    String value = redis.get(key);
    // 如果value为空， 则开始重构缓存
    if (value == null) {
        // 只允许一个线程重建缓存， 使用nx， 并设置过期时间ex
        String mutexKey = "mutext:key:" + key;
        if (redis.set(mutexKey, "1", "ex 180", "nx")) {
             // 从数据源获取数据
            value = db.get(key);
            // 回写Redis， 并设置过期时间
            redis.setex(key, timeout, value);
            // 删除key_mutex
            redis.delete(mutexKey);
        }// 其他线程休息50毫秒后重试
        else {
            Thread.sleep(50);
            get(key);
        }
    }
    return value;
}
```



# 5 **缓存与数据库不一致** 

## 5.1 定义

在大并发下，同时操作数据库与缓存会存在数据不一致性问题

## 5.2 产生原因

- 双写不一致情况 。

  我们去写redis的时候，基本都是先写入数据库，然后再写入redis。出现问题的场景如下：

  1. 线程1写入数据库后，一些原因卡住了
  2. 线程2也去写数据库，更新缓存
  3. 线程1再去更新缓存的时候，是有问题的，因为他的数据不是最新的。

  ![image-20230228225410250](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228225410250.png)

- 读写并发不一致 

  我们还有种写入redis的方式是，写完数据库后删除缓存，查询数据的时候，再去更新缓存。出现问题的场景如下：

  1. 线程1更新数据库为10，删除缓存

  2. 线程3查询缓存，此时数据库的值为10

  3. 线程2更新数据库为6，删除缓存

  4. 线程3更新缓存为10

  

  ![image-20230228225417391](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228225417391.png)



## 5.3 解决方案

### 5.3.1 分布式读写锁

- 对于并发几率很小的数据(如个人维度的订单数据、用户数据等)，这种几乎不用考虑这个问题，很少会发生缓存不一致，可以给缓存数据加上过期时间，每隔一段时间触发读的主动更新即可。

- 如果不能容忍缓存数据不一致，可以通过加**分布式读写锁**保证并发读写或写写的时候按顺序排好队，**读的时候相当于无锁**，redsion就支持。
- **放入缓存的数据应该是对实时性、一致性要求不是很高的数据。**如果**写多读多**的情况又不能容忍缓存数据不一致，那就没必要加缓存了，可以直接操作数据库。如果数据库抗不住压力，还可以把缓存作为数据读写的主存储，异步将数据同步到数据库，数据库只是作为数据的备份。 

![image-20230228225427756](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20230228225427756.png)
