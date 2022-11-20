---
title: Redis集群三种方式
date: 2021-12-28 22:45:20
tags: redis主从架构
categories: Redis
---

# 为什么要有集群

- 单个Redis存在不稳定性，当Redis服务宕机或硬盘故障，系统崩溃之后，就没有可用的服务了，还会造成数据的丢失
- 单个Redis的读写能力也是有限的
- 还由于互联网的三高[架构](https://so.csdn.net/so/search?q=架构&spm=1001.2101.3001.7020)，高并发，高性能，高可用

> 通过添加服务器的数量，提供相同的服务，从而让服务器达到一个稳定，高效的状态



# Redis集群的三种模式

## 1. 主从模式

### 产生的原因

- 避免单点Redis服务器故障。数据丢失，可能对业务造成灾难性打击
- 容量瓶颈。内存不足，一台redis内存是有限的

### 特征

- 一个master可以拥有多个slave，一个slave只对应一个master
- **一主多从，主负责写，并且将数据复制到其它的 slave 节点，从节点负责读**。

- 所有的读请求全部走从节点。这样也可以很轻松实现水平扩容，支撑**读高并发**。
- slave挂了不影响其他slave的读和master的读和写，重新启动后会将数据从master同步过来。可以使用全量复制和部分复制
- master挂了以后，不影响slave的读，但redis不再提供写服务，master重启后redis将重新对外提供写服务
- master挂了以后，不会在slave节点中重新选一个master，只能手动的一个一个改，不合适，就有了之后的哨兵



<img src="https://tva1.sinaimg.cn/large/008i3skNly1gxv2a0o0mqj30oi0lsgmm.jpg" style="zoom:30%;" />

### **Redis主从复制工作原理**

#### 1. 全量复制流程：

Redis全量复制一般发生在Slave初始化阶段，这时Slave需要将Master上的所有数据都复制一份，具体步骤如下：

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gxv2yicirhj30tq0lggnx.jpg" style="zoom:50%;" />

1. 如果你为master配置了一个slave，不管这个slave是否是第一次连接上Master，它都会发送一个**SYNC**命令(redis2.8版本之前的命令)master请求复制数据。
2. master收到SYNC命令后，会在后台进行数据持久化通过bgsave生成最新的rdb快照文件。
3. master将生成的rdb数据发给slave。
4. Master由于从执行bgsave到生成rdb数据存在时间，在此时间间隔内，可能有新的客户端请求，master会把这些最近的请求缓存在内存中。（也就是一个最近数据的缓冲区，默认大小为1M）
5. 当第3步的rdb数据发完后，他会把这些缓存的数据也发给slave
6. 当slave有老的rdb数据，就会先把老的数据清掉。重新生成包含缓存的rd b数据，并加载到内存中。
7. 之后，master和slave通过socket长连接，持续进行命令同步，从而保证主从数据一致。

> 当master与slave之间的连接由于某些原因而断开时，slave能够自动重连Master。

> 如果master收到了多个slave并发连接请求，它只会进行一次持久化，而不是一个连接一次，然后再把这一份持久化的数据发送给多个并发连接的slave。  



**当master和slave断开重连后，一般都会对整份数据进行复制。但从redis2.8版本开始，master和slave断开重连后支持部分复制**。



#### 2. 部分复制流程

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gxv2xkkaizj318w0ro78b.jpg" style="zoom: 33%;" />

1. 从2.8版本开始，slave与master能够在网络连接断开**重连后只进行部分数据复制。** 
2. master会在其内存中创建一个复制数据用的缓存队列，缓存最近一段时间的数据。master和它所有的slave都维护了复制的数据下标offset和master的进程id。
3. 当网络连接断开后，slave会请求master继续进行未完成的复制。
   - 从节点数据下标 offset还在缓存队列里，那么将会从slave记录的数据下标开始从缓存区复制。
   - 如果master进程id变化了，或者从节点数据下标 offset太旧，已经不在master的缓存队列里了，那么将会进行一次全量数据的复制
   

#### 3 增量复制

- 是指在初始化的全量复制并开始正常工作之后，master服务器将发生的写操作同步到slave服务器的过程

- 增量复制的过程主要是master服务器每执行一个写命令就会向slave服务器发送相同的写命令，slave服务器接收并执行收到的写命令。

## 2. 哨兵模式

###  产生的原因

主从模式下，主机宕机，需要人为去设置master，哨兵模式就是解决：不用人为进行干预，高可用的恢复正常。

> 哨兵（sentinel）是一个分布式系统，用于对主从结构中的每台服务器进行监控，当出现故障时通过投票机制选择新的master并将所有slave连接到新的master

### 特征

- 监控：sentinel哨兵是特殊的redis服务，不提供读写服务，主要用来不断地检查master和slave是否正常运行。 
- 自动故障转移：当master出现问题时，选取一个slave作为master，将其他slave连接到新地master，并告知客户端新地服务器地址
- 通知：当redis的主节点发生变化，哨兵会第一时间感知到，并且将新的redis主节点通知给client端(这里面redis的client端一般都实现了订阅功能，订阅sentinel发布的节点变动消息) 

**注⚠️：**

- 哨兵架构下client端第一次从哨兵找出redis的主节点，后续就直接访问redis的主节点，不会每次都通过sentinel代理访问redis的主节点。
- 哨兵也是一台redis服务器，只是不提供数据服务
- 为了高可用一般都推荐至少部署三个哨兵节点

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gxv33qmtvpj319e0pktbv.jpg" style="zoom:33%;" />

### **哨兵leader选举流程** 

当一个master服务器被某sentinel视为下线状态后，该sentinel会与其他sentinel协商选出sentinel的leader进行故障转移工作。

1. 每个发现master服务器进入下线的sentinel都可以要求其他sentinel选自己为sentinel的 leader，选举是先到先得。
2. 超过一半的sentinel选举某sentinel作为leader。之后该sentinel进行故障转移操作，从存活的slave中选举出新的master

## 3. Cluster模式

### 产生的原因

- 哨兵模式，如果master节点异常，在主从切换的瞬间，存在不可用的情况。
- 哨兵模式只有单个主节点可以用来写，没法支持很高的并发。
- 哨兵模式单个节点的内存也不宜设的太大，会导致持久化文件过大，影响数据恢复主从效率。

### 特征

- Redis-Cluster采用无中心结构，每个节点保存数据和整个集群状态。
- 提供多个master节点提供写服务，每个master节点中存储的数据都不一样，这些数据通过数据分片的方式被自动分割到不同的master节点上。
- 每个master节点下面还需要添加至少1个slave节点，这样当某个master节点发生故障后，可以从它的slave节点中选举一个作为新的master节点继续提供服务
- Redis Cluster 将所有数据划分为 16384 个 slots(槽位)，每个节点负责其中一部分槽位。槽位的信息存储于每 个节点中。当 Redis Cluster 的客户端来连接集群时，它也会得到一份集群的槽位配置信息并将其缓存在客户端本地。这样当客户端要查找某个 key 时，可以直接定位到目标节点。



<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h6501y1ohvj21fo0sadjh.jpg" style="zoom:33%;" />

### **槽位定位算法** 

Cluster 默认会对 key 值使用 crc16 算法进行 hash 得到一个整数值，然后用这个整数值对 16384 进行取模 来得到具体槽位。 

`HASH_SLOT = CRC16(key) mod 16384` 



### **网络抖动** 

- Redis Cluster 提供了一种选项`cluster-­node-­timeout`，表示当某个节点持续 timeout 的时间失联时，才可以认定该节点出现故障，需要进行主从切换。

- 如果没有这个选项，网络抖动会导致主从频 繁切换 (数据的重新复制)。



### **Redis集群选举原理分析** 

当slave发现自己的master变为FAIL状态时，便尝试进行Failover，以期成为新的master。由于挂掉的master可能会有多个slave，从而存在多个slave竞争成为master节点的过程， 其过程如下： 

1. slave发现自己的master变为FAIL 

2. 将自己记录的集群currentEpoch加1，并广播FAILOVER_AUTH_REQUEST 信息

3. 其他节点收到该信息，只有master响应，判断请求者的合法性，并发送FAILOVER_AUTH_ACK，对每一个 slaver只发送一次ack 

4. slave收到超过半数master的ack后变成新Master
5. .slave广播Pong消息通知其他集群节点。

注意⚠️：

- 从节点并不是在主节点一进入 FAIL 状态就马上尝试发起选举，而是有一定延迟。一定的延迟确保我们等待 FAIL状态在集群中传播，slave如果立即尝试选举，其它masters或许尚未意识到FAIL状态，可能会拒绝投票
- 延迟计算公式： DELAY = 500ms + random(0 ~ 500ms) + SLAVE_RANK * 1000ms 

​       SLAVE_RANK表示此slave已经从master复制数据的总量的rank。Rank越小代表已复制的数据越新。这种方式下，持有最新数据的slave将会首先发起选举（理论上）。



### **集群脑裂数据丢失问题** 

网络分区导致脑裂后多个主节点对外提供写服务，一旦网络分区恢复， 会将其中一个主节点变为从节点，这时会有大量数据丢失。

规避方法可以在redis配置里加上参数(这种方法不可能百分百避免数据丢失，参考集群leader选举机制）

```java
min‐replicas‐to‐write 1 //写数据成功最少同步的slave数量，这个数量可以模仿大于半数机制配置，比如 集群总共三个节点可以配置1，加上leader就是2，超过了半数
```

这个配置在一定程度上会影响集群的可用性，比如slave要是少于1个，这个集群就算leader正常也不能提供服务了，需要具体场景权衡选择。 



### **Redis集群为什么至少需要三个master节点，并且推荐节点数为奇数**

- 因为新master的选举需要大于半数的集群master节点同意才能选举成功
- 奇数个master节点可以在满足选举该条件的基础上节省一个节点，比如三个master节点和四个master节点的集群相比，都只允许挂一个节点，否则集群不可用。所以奇数的master节点更多的是**从节省机器资源角度出发**说的。 



### **Redis集群对批量操作命令的支持** 

对于类似mset，mget这样的多个key的原生批量操作命令，redis集群只支持所有key落在同一slot的情况，如 果有多个key一定要用mset命令在redis集群上操作，则可以在key的前面加上{XX}，这样参数数据分片hash计算的只会是大括号里的值，这样能确保不同的key能落到同一slot里去，示例如下： 

```java
 mset {user1}:1:name zhuge {user1}:1:age 18
```

假设name和age计算的hash slot值不一样，但是这条命令在集群下执行，redis只会用大括号里的 user1 做 hash slot计算，所以算出来的slot值肯定相同，最后都能落在同一slot
