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



# 4 Redis热key问题 

## 4.1定义

在Redis中，我们把访问频率高的Key，称为热Key（例如一个热门的娱乐新闻））。热key问题就是，突然有几十万的请求去访问redis上的某个特定key。那么这样会造成redis服务器短时间流量过于集中，很可能导致redis的服务器宕机。那么接下来对这个Key的请求，都会直接请求到我们的后端数据库中，数据库性能本来就不高，这样就可能直接压垮数据库，进而导致后端服务不可用。

## 4.2 产生原因

用户消费的数据远大于生产的数据，如商品秒杀、热点新闻、热点评论等读多写少的场景

双十一秒杀商品，短时间内某个爆款商品可能被点击/购买上百万次，或者某条爆炸性新闻等被大量浏览，此时会造成一个较大的请求Redis量，这种情况下就会造成热点Key问题。

## 4.3 怎么发现热key

1. 凭借业务经验，进行预估哪些是热key
   其实这个方法还是挺有可行性的。比如某商品在做秒杀，那这个商品的key就可以判断出是热key。缺点很明显，并非所有业务都能预估出哪些key是热key。

2. *在客户端进行收集*。

   操作redis之前，加入一行代码进行数据统计。那么这个数据统计的方式有很多种，也可以是给外部的通讯系统发送一个通知信息。缺点就是对客户端代码造成入侵。

3. 用redis自带命令

   (1). monitor命令，该命令可以实时抓取出redis服务器接收到的命令，然后写代码统计出热key是啥。当然，也有现成的分析工具可以给你使用，比如`redis-faina`。但是该命令在高并发的条件下，有内存增暴增的隐患，还会降低redis的性能。
   (2). hotkeys参数，redis 4.0.3提供了redis-cli的热点key发现功能，执行redis-cli时加上–hotkeys选项即可。但是该参数在执行的时候，如果key比较多，执行起来比较慢。

4. *自己抓包评估*

   自己写程序监听端口，按照RESP协议规则解析数据，进行分析。缺点就是开发成本高，维护困难，有丢包可能性。

## 4.4 如何解决

1. Redis集群扩容：增加分片副本，分摊客户端发过来的读请求；**
2. 使用二级缓存，即JVM本地缓存，减少Redis的读请求。比如利用`HashMap`。在你发现热key以后，把热key加载到系统的JVM中。针对这种热key请求，会直接从jvm中取，而不会走到redis层。

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

## 5.2 缓存一致性的解决方案

### 5.2.1 合理设置缓存过期时间

对于并发几率很小的数据(如个人维度的订单数据、用户数据等)，这种几乎不用考虑这个问题，很少会发生缓存不一致，可以给缓存数据加上过期时间，每隔一段时间触发读的主动更新即可。

### 5.2.2 **异步消息队列**

通过异步消息队列（如Kafka、RabbitMQ等）来同步更新缓存和数据库数据。在数据更新时，将更新操作发送到消息队列，消费者监听消息队列，更新缓存和数据库。

注意：需要保证消息的[可靠性](https://pyr9.github.io/kafka%E7%9A%84%E6%B6%88%E6%81%AF%E9%9B%B6%E4%B8%A2%E5%A4%B1%E6%96%B9%E6%A1%88/)、[顺序性](https://pyr9.github.io/kafka%E5%B8%B8%E8%A7%81%E9%9D%A2%E8%AF%95%E9%A2%98/#%E6%B6%88%E8%B4%B9%E9%A1%BA%E5%BA%8F%E6%80%8E%E4%B9%88%E4%BF%9D%E8%AF%81%EF%BC%9F)、[幂等性](https://pyr9.github.io/kafka%E5%B8%B8%E8%A7%81%E9%9D%A2%E8%AF%95%E9%A2%98/#kafka-%E5%A6%82%E4%BD%95%E4%B8%8D%E6%B6%88%E8%B4%B9%E9%87%8D%E5%A4%8D%E6%95%B0%E6%8D%AE%EF%BC%9F)

###  5.2.2 **双写策略**

在更新数据库的同时，同步更新缓存。需要注意的是，在分布式系统中，双写策略需要确保事务的一致性。

Spring中试用注解：

**@Cacheable**：标记的方法会先检查缓存是否存在数据，如果存在则直接返回缓存数据，如果不存在则执行方法并将返回结果缓存起来。

**@CacheEvict**：标记的方法在执行后，会从缓存中移除对应的缓存数据。

**@CachePut**：标记的方法在执行后，会将方法的返回值更新到缓存中。

**@Transactional**：确保数据库操作的原子性，防止在操作数据库时出现并发问题。

### 5.2.1 **使用分布式锁**

在高并发场景下，使用分布式锁（如Redis，zookeeper分布式锁）来控制缓存的更新操作，确保数据的一致性。

### 5.5.4 **监控和日志**

通过监控和日志系统，及时发现缓存和数据库不一致的问题，并进行修复，调用幂等性接口。



注：**放入缓存的数据应该是对实时性、一致性要求不是很高的数据。**如果**写多读多**的情况又不能容忍缓存数据不一致，那就没必要加缓存了，可以直接操作数据库。如果数据库抗不住压力，还可以把缓存作为数据读写的主存储，异步将数据同步到数据库，数据库只是作为数据的备份。 

