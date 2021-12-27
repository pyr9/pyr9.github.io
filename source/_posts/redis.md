---
title: redis 核心数据结构
date: 2021-12-26 19:07:28
tags: redis 核心数据结构
categories: redis
---

# redis 是什么？

> redis是一个高性能的key-value 数据库，它可以用来存储字符串，哈希，列表，集合，有序集合。

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gxrgdfb02gj30tq0v2whf.jpg" style="zoom:50%;" />

# String

String 是redis中最基础的数据结构，主要用在常规计数，如：统计网站访问数据量，当前在线人数等

**应用场景**

- 单值缓存

  ```sql
  127.0.0.1:6379> set key1 "zhangsan"
  OK
  127.0.0.1:6379> get key1
  "zhangsan"
  ```

- 计数器

  ```sql
  127.0.0.1:6379> set article:readcount:mysql-tutorial 12
  OK
  127.0.0.1:6379> get article:readcount:mysql-tutorial
  "12"
  127.0.0.1:6379> incr article:readcount:mysql-tutorial
  (integer) 13
  127.0.0.1:6379> get article:readcount:mysql-tutorial
  "13"
  ```

- 分布式系统全局序列号	
  INCRBY  orderId  1000		//redis批量生成序列号提升性能

  ```sql
  127.0.0.1:6379> get orderId
  "99"
  127.0.0.1:6379> incrby orderId  1000
  (integer) 1099
  ```

- 分布式锁
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

# hash

- 定义：Hash是一个field 和value 的映射表，特别适合存储对象，比如存储用户信息，商品信息等。
- 优缺点：
  - 优点
    - 同类数据归类整合储存，方便数据管理
    - 相比string操作消耗内存与cpu更小
    - 相比string储存更节省空间
  - 缺点
    - 过期功能不能使用在field上，只能用在key上

**应用场景**

- 对象缓存

```sql
127.0.0.1:6379> HMSET  user  1:name  zhuge 1:balance  1888
OK
127.0.0.1:6379> HMGET  user  1:name  1:balance
1) "zhuge"
2) "1888"
```

- 电商购物车
  - 以用户id为key；     商品id为field；    商品数量为value；
  - 购物车操作
    - 添加商品 ` hset cart:user2 "apple" 1`
    - 增加数量  ` hincrby cart:user2 "apple" 3`
    - 商品总数 `hlen cart:user2`
    - 删除商品 `hdel cart:user2 "apple"`
    - 获取购物车所有商品 `hgetall cart:user2`

# list

list是一个链表结构，主要功能是push和pop（添加和弹出集合元素），获取一个范围的值等。常用于：粉丝列表，最新消息排行。

**应用场景**

- 微博消息和微信公号消息
  张三关注了2 个订阅号，美食专栏和娱乐周边

  - 美食专栏发推送

     `lpush msg:zhangSan "apple is yammy!"`

  - 娱乐周边发推送.

    `lpush msg:zhangSan "YangMi is beautiful!"`

  - 查看张三关注的最新订阅号消息.

     `lrange msg:zhangSan 0 5`

# set

Redis 中的set集合是无序且不可重复的，它最大的优势可以进行交集，丙级，差集等，常用来求共同好友等。

**应用场景**

- 微信抽奖小程序

  - 点击参与抽奖加入集合 

  ​      `sadd iphone13 zhangsan`

  - 查看参与抽奖所有用户

  ​      `smembers iphone13`	  

  - 抽取count名中奖者  

    - 返回集合中2个随机数.

      `srandmember iphone13 2`

    - 移除并返回集合中的2个随机元素 

       `spop iphone13 2`

- 微信微博点赞，收藏，标签

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

- 集合操作

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

# zset

在set的基础上，增加了一个权重系数score，使集合中的元素可以按照score进行有序排列，常用于实现排行榜。

**应用场景**

- Zset集合操作实现排行榜

  - 点击新闻   

    ZINCRBY  hotNews:20190819  1  守护香港

  - 展示当日排行前十 

     ZREVRANGE  hotNews:20190819  0  10  WITHSCORES 

  - 七日搜索榜单计算
    ZUNIONSTORE  hotNews:20190813-20190819  7 
    hotNews:20190813  hotNews:20190814... hotNews:20190819

    > [ZUNIONSTORE destination numkeys key [key ...\]](https://www.runoob.com/redis/sorted-sets-zunionstore.html)
    > 计算给定的一个或多个有序集的并集，并存储在新的 key 中

  - 展示七日排行前十
    ZREVRANGE hotNews:20190813-20190819  0  10  WITHSCORES

    > [ ZREVRANGE key start stop [WITHSCORES\]](https://www.runoob.com/redis/sorted-sets-zrevrange.html) 返回有序集中指定区间内的成员，通过索引，分数从高到低

