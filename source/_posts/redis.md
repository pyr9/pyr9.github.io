---
title: redis常见面试题
date: 2021-12-26 19:07:28
tags: redis 核心数据结构
categories: Redis
---

> redis是一个高性能的key-value 数据库，它可以用来存储字符串，哈希，列表，集合，有序集合。

# 1. redis常见数据结构

## 1 String

String 是redis中最基础的数据结构，主要用在常规计数，如：统计网站访问数据量，当前在线人数等

### 1.**应用场景**

#### 1 单值缓存

```sql
127.0.0.1:6379> set key1 "zhangsan"
```

#### 2 计数器

```sql
127.0.0.1:6379> set article:readcount:mysql-tutorial 12
```

#### 3. 分布式系统全局序列号	
INCRBY  orderId  1000		//redis批量生成序列号提升性能

```sql
127.0.0.1:6379> incrby orderId  1000
```

#### 4. 分布式锁(redisson)

- SETNX  product:10001   
- 执行业务操作。。。
- DEL  product:10001。                                     //执行完业务释放锁
- SET product:10001 true  ex  10  nx	          //防止程序意外终止导致死

> SETNX（ **SET** if **N**ot e**X**ists ）
>
> - 在指定的 key 不存在时，为 key 设置指定的值，这种情况下等同 [SET](https://www.redis.com.cn/commands/set.html) 命令。当 `key`存在时，什么也不做。
>
> - 返回值
>   - `1` 如果key被设置了
>   - `0` 如果key没有被设置

## 2. hash

Hash是一个field 和value 的映射表，特别适合存储对象，比如存储用户信息，商品信息等。

### 1. 优缺点：

- 优点
  - 同类数据归类整合储存，方便数据管理
  - 相比string操作消耗内存与cpu更小
  - 相比string储存更节省空间
- 缺点
  - 过期功能不能使用在field上，只能用在key上

### 2. **应用场景**

#### 1 对象缓存

```sql
127.0.0.1:6379> HMSET  user  1:name  zhuge 1:balance  1888
```

#### 2 电商购物车

- 以用户id为key；     商品id为field；    商品数量为value；
- 购物车操作
  - 添加商品 ` hset cart:user2 "apple" 1`
  - 增加数量  ` hincrby cart:user2 "apple" 3`
  - 商品总数 `hlen cart:user2`
  - 删除商品 `hdel cart:user2 "apple"`
  - 获取购物车所有商品 `hgetall cart:user2`

## 3. list

list是一个链表结构，主要功能是push和pop（添加和弹出集合元素），获取一个范围的值等。常用于：粉丝列表，最新消息排行。

### 1. **应用场景**

#### 1. 微博消息和微信公号消息
张三关注了2 个订阅号，美食专栏和娱乐周边

- 美食专栏发推送

   `lpush msg:zhangSan "apple is yammy!"`

- 娱乐周边发推送.

  `lpush msg:zhangSan "YangMi is beautiful!"`

- 查看张三关注的最新订阅号消息.

   `lrange msg:zhangSan 0 5`

## 4 set

Redis 中的set集合是无序且不可重复的，它最大的优势可以进行交集，丙级，差集等，常用来求共同好友等。

### 1. **应用场景**

#### 1  微信抽奖小程序

- 点击参与抽奖加入集合 

​      `sadd iphone13 zhangsan`

- 查看参与抽奖所有用户

​      `smembers iphone13`	  

- 抽取count名中奖者  

  - 返回集合中2个随机数.

    `srandmember iphone13 2`

  - 移除并返回集合中的2个随机元素 

     `spop iphone13 2`

#### 2. 微信微博点赞，收藏，标签

- 点赞
  SADD  like:{消息ID}  {用户ID}
- 取消点赞
  SREM like:{消息ID}  {用户ID}
- 检查用户是否点过赞
  SISMEMBER  like:{消息ID}  {用户ID}
- 获取点赞的用户列表
  SMEMBERS like:{消息ID}
- 获取点赞用户数 
  SCARD like:{消息ID}

### 2.集合操作

- 交集：SINTER set1 set2 set3 
- 并集：SUNION set1 set2 set3
- 差集：SDIFF set1 set2 set3 

```sql
127.0.0.1:6379> sadd set1 1 2 3 4
(integer) 4
127.0.0.1:6379> sadd set2 1 3 5 6
(integer) 4
127.0.0.1:6379> sinter set1 set2
1) "1"
2) "3"
127.0.0.1:6379> sunion set1 set2
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"
127.0.0.1:6379> sdiff set1 set2
1) "2"
2) "4"
```

##  5 zset

在set的基础上，增加了一个权重系数score，使集合中的元素可以按照score进行有序排列，常用于实现排行榜。

### 1. **应用场景**

#### 1. Zset集合操作实现排行榜

- 点击新闻   

  ```
  ZINCRBY  hotNews:20190819  1  守护香港
  ```

- 展示当日排行前十 

   ```
   ZREVRANGE  hotNews:20190819  0  10  WITHSCORES 
   ```

- 七日搜索榜单计算
  
  ```
  ZUNIONSTORE  hotNews:20190813-20190819  7 
  hotNews:20190813  hotNews:20190814... hotNews:20190819
  ```

  > [ZUNIONSTORE destination numkeys key [key ...\]](https://www.runoob.com/redis/sorted-sets-zunionstore.html)
  > 计算给定的一个或多个有序集的并集，并存储在新的 key 中
  
- 展示七日排行前十
  
  ```ZREVRANGE hotNews:20190813-20190819  0  10  WITHSCORES```
  
  > [ ZREVRANGE key start stop [WITHSCORES\]](https://www.runoob.com/redis/sorted-sets-zrevrange.html) 返回有序集中指定区间内的成员，通过索引，分数从高到低

# 2. redis 为什么快

Redis 是一个高性能的键值存储数据库，以其快速的读写速度而著称。其速度主要得益于以下几个关键因素：

### 1. 内存存储

Redis 将所有数据存储在内存中（RAM），而不是传统数据库那样存储在磁盘上。内存的读写速度远远快于磁盘，因此 Redis 的操作速度非常快。

### 2. 简单的数据结构

Redis 提供了一些简单且高效的数据结构，例如字符串、哈希、列表、集合、有序集合、位图、HyperLogLog、地理空间索引和流。每种数据结构都经过优化，可以快速地执行常见操作。

### 3. 单线程架构

Redis 使用单线程模型处理客户端请求。这种设计避免了多线程环境中的上下文切换和竞态条件，从而减少了复杂性和开销。尽管单线程可能看起来是个瓶颈，但由于 Redis 的核心操作非常快且大部分是在内存中完成的，因此单线程模型在大多数情况下性能非常高。

### 4. 非阻塞 I/O

Redis 使用了基于事件驱动的 I/O 多路复用机制（如 epoll 和 kqueue），使得它能够同时处理大量客户端连接而不会出现阻塞。这种非阻塞 I/O 模型使得 Redis 能够有效地处理并发请求。

### 5. 紧凑的内存管理

Redis 对内存使用进行了高度优化。例如，它使用了一种高效的内存分配机制来最小化内存碎片，并提供了一些数据压缩选项（如 ziplist 和 quicklist）来节省内存空间。

### 6. 高效的序列化和反序列化

Redis 的 RDB 和 AOF 两种持久化机制都经过优化，可以快速地将数据从内存保存到磁盘并恢复。这种高效的序列化和反序列化过程减少了持久化操作对性能的影响。

### 7. 高效的网络协议

Redis 使用了一个简单、高效的二进制协议 RESP（Redis Serialization Protocol），来进行客户端和服务器之间的数据交换。这个协议设计简单，解析速度非常快，进一步提高了 Redis 的性能。

### 8. 本地化的数据处理

Redis 提供了丰富的原子操作，可以在服务器端完成复杂的数据处理任务，减少了网络往返次数。例如，可以在 Redis 内部执行复杂的集合操作，而不需要将数据传输到客户端进行处理。

### 9. 数据分片（Sharding）

虽然 Redis 本身是单线程的，但它支持数据分片（sharding），即将数据分布在多个 Redis 实例上。通过这种方式，Redis 能够水平扩展，处理更大的数据量和更高的并发请求。

# 3. Redis 内存淘汰策略 

在 Redis 中，淘汰策略（也称为内存回收策略）决定了在 Redis 实例达到内存限制时，应该如何处理新写入的数据。主要包括以下几种：

- **noeviction**：对于写请求不再提供服务，会直接返回错误。这是**默认策略**。
- **allkeys-random**：从**所有key**中**随机**移除某个键。
- **volatile-random**：从**设置了过期时间的键**中**随机**移除某个键。
- **volatile-ttl**：从**设置了过期时间的键**中移除**剩余寿命最短的键**（即将过期的键）。

- **allkeys-lru**：使用LRU（Least Recently Used，最近最少使用）算法，从**所有key**中选取**使用最少**的进行淘汰。

- **volatile-lru**：使用LRU（Least Recently Used，最近最少使用）算法，从设**置了过期时间的key**中移除**最近最少使用**的键。
- **allkeys-lfu:** 使用LFU（Least Frequently Used，最不经常使用），从**所有key中**选择**某段时间之内使用频次最少**的键值对清除
- **volatile-lfu**: 使用LFU（Least Frequently Used，最不经常使用），从**设置了过期时间的key**中选择**某段时间之内使用频次最小**的键值对清除掉

> Redis 的淘汰算法有多种:
>
> - 随机
> - TTL: 从设置了过期时间的 Keys 中获取最早过期的 一批 Keys
> - LRU (Least Recently Used) 算法: 所有的 Keys 都根据最后被访问的时间来进行排序的，所以在淘汰时只需要按照所有 Keys 的最后被访问时间，由小到大来进行即可
> - LFU（Least Frequently Used）算法: 照被访问次数降序排列，访问次数相同的按最近访问时间降序排列，链表满的时候从链表尾部移出数据。

# 4. Redis 常见使用场景

1. 缓存: DB缓存，减轻DB服务器压力, 提高系统响应。需保证缓存内容与数据库的一致性
2. 分布式锁：利用Redis的`SETNX key value`这个命令获取锁，并设置过期时间。需处理：a.增加ID的复杂性，满足安全性 b.合理设置一个超时时间, 不要太长或太短，还需考虑锁续命。推荐使用第三方库`redisson`
3. 分布式ID：利用Redis的原子性递增操作，生成分布式唯一ID。优点速度快，缺点需保证redis的高可用和持久化
4. 计数器：利用incr方法，实现文章的阅读量，微博点赞数，IP限流等
5. 排行榜：利用zset, 记录条目的同时，记录成绩
6. 实现消息队列：利用list/set
