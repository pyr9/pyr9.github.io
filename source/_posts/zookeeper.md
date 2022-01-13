
---

title: zookeeper特性与节点介绍
date: 2021-09-22 22:43:51
tags: 

-  分布式框架

categories:  zookeeper

---

# zookeeper

##  产生背景

1. 单体向分布式服务转变，会产生多个节点间协同问题，如：

   > - 假设有3个节点，1个job，该job该由哪个节点执行？
   >
   > - 若该job由节点1执行，要是1挂了，现在该2还是3执行？

2. 上游和下游服务的发现，如：

   > * 上游挂了，下游怎么知道？
   > * 下游服务新增，上游怎么知道？

3. 多节点的协调问题，如：并发产生了请求，该怎么保证请求的幂等性？


> **由于节点自身协调不可靠，性能不高，故需要一个独立服务来做协调，他必须可靠且保证性能。**

##  概要

1. zookeeper是用于分布式服务的协调服务。
2. 它对外公布了一组简单的API，分布式应用程序可以基于这些API，用于同步节点状态，节点配置，服务注册等信息
3. 它支持java和c两种语言。

## 工作机制

从设计模式的角度理解，zookeeper基于观察者模式。他负责存储和管路大家都关心的数据，然后接受观察，一旦数据状态发生变化，zookeeper就通知注册者。

## 特点

1. zookeeper是由一个leader和多个follower组成的集群
2. 半数以上节点存活，zookeeper 即可以正常服务
3. 每个server的数据一致
4. 来自同一个client的请求依次执行
5. 原子性，数据更新要么成功，要么失败
6. zookeeper数据量少，故同步快，一定时间内，client可以读到最新数据

 

## zookeeper 启动

- 官网下载最新版本 [Apache Downloads](https://www.apache.org/dyn/closer.lua/zookeeper/zookeeper-3.7.0/apache-zookeeper-3.7.0-bin.tar.gz)，并解压
- 进入conf目录，拷贝默认配置  `cp zoo_sample.cfg   zoo.cfg`
- 进入bin目录，启动服务端`./zkServer.sh start`
- 进入bin目录，进入客户端`./zkCli.sh`



## 常规配置文件说明

> tickTime=2000   #zookeeper时间配置中的基本单位 (毫秒) 

>initLimit=10     #允许follower初始化连接到leader最大时长，它表示tickTime时间倍数 即:initLimit*tickTime

> syncLimit=5 	#允许follower与leader数据同步最大时长,它表示tickTime时间倍数

>dataDir=/tmp/zookeeper  #zookeper 数据存储目录

>clientPort=2181  #对客户端提供的端口号

>maxClientCnxns=60  #单个客户端与zookeeper最大并发连接数

>autopurge.snapRetainCount=3. #保存的数据快照数量，之外的将会被清除

>autopurge.purgeInterval=1  #自动触发清除任务时间间隔，小时为单位。默认为0，表示不自动清除。



## 客户端命令

- close     关闭当前会话
- connect host:port        重新连接指定Zookeeper服务

- create [-s] [-e] [-c] [-t ttl] path [data] [acl]          创建节点
- delete [-v version] path             删除节点，(不能存在子节点） 
- deleteall path [-b batch size]      删除路径及所有子节点
- get [-s] [-w] path.        查看节点数据 -s 包含节点状态 -w 添加监听 
- getAcl [-s] path.      列出子节点 -s状态 -R 递归查看所有子节点 -w 添加监听
- history     查看执行的历史记录
- redo cmdno  重复 执行命令，history 中命令编号确定
- quit  退出客户端

```sql
[zk: localhost:2181(CONNECTED) 1] ls /
[zookeeper]
[zk: localhost:2181(CONNECTED) 2] create /node1
Created /node1
[zk: localhost:2181(CONNECTED) 3] get /node1
null
[zk: localhost:2181(CONNECTED) 4] set /node1 "node1 value"
[zk: localhost:2181(CONNECTED) 5] get /node1
node1 value
[zk: localhost:2181(CONNECTED) 6] create /node1/child1
Created /node1/child1
[zk: localhost:2181(CONNECTED) 7] set  /node1/child1 "child1"
[zk: localhost:2181(CONNECTED) 8] get /node1/child1 "child1"
'get path [watch]' has been deprecated. Please use 'get [-s] [-w] path' instead.
child1
[zk: localhost:2181(CONNECTED) 9] get /node1/child1
child1
[zk: localhost:2181(CONNECTED) 10] delete /node1
Node not empty: /node1
[zk: localhost:2181(CONNECTED) 11] deleteall /node1
WATCHER::
WatchedEvent state:SyncConnected type:NodeDeleted path:/node1/child1
[zk: localhost:2181(CONNECTED) 12] ls /
[zookeeper]
[zk: localhost:2181(CONNECTED) 13] history
3 - get /node1
4 - set /node1 "node1 value"
5 - get /node1
6 - create /node1/child1
7 - set  /node1/child1 "child1"
8 - get /node1/child1 "child1"
9 - get /node1/child1
10 - delete /node1
11 - deleteall /node1
12 - ls /
13 - history

[zk: localhost:2181(CONNECTED) 16] redo 2
Created /node1
[zk: localhost:2181(CONNECTED) 17] ls /
[node1, zookeeper]
[zk: localhost:2181(CONNECTED) 18] get -s /node1
null
cZxid = 0x8
ctime = Thu Jan 13 22:49:37 CST 2022
mZxid = 0x8
mtime = Thu Jan 13 22:49:37 CST 2022
pZxid = 0x8
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 0
```

## 节点介绍

### 数据基本单元

* zookeepe中的基本数据单元是——节点（znode）。节点之下可以包含子节点，最后以树级方式呈现。

- 每个节点拥有唯一的路径path.
- 客户端基于path上传节点数据，zookeepe收到数据后，会通知监听该节点的客户端（一个或多个）。
- 之前，各个节点的数据是相互孤立的（也可以自己同步，但是没有zookeeper性能高，可靠）。现在，有了zookeeper,各节点的数据可以通过zookeeper可以得到一个高性能高可靠的同步。

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gycegg0aihj30u00v0wgk.jpg" style="zoom:50%;" />

### 节点的结构

* **path**:唯一路径 
* **childNode**：子节点
* **stat**:状态属性:即path对应的值
* **type**:节点类型

### 节点类型

| 类型                  | 描述                           |
| :-------------------- | :----------------------------- |
| PERSISTENT            | 持久节点                       |
| PERSISTENT_SEQUENTIAL | 持久序号节点                   |
| EPHEMERAL             | 临时节点(不可在拥有子节点)     |
| EPHEMERAL_SEQUENTIAL  | 临时序号节点(不可在拥有子节点) |

#### PERSISTENT（持久节点）

- 持久化保存的节点，也是默认创建的

```sql
create /test
```

#### PERSISTENT_SEQUENTIAL(持久序号节点)

创建时zookeeper 会在路径上加上序号作为后缀，。非常适合用于分布式锁、分布式选举等场景。创建时添加 -s 参数即可。

```sql
create -s  /temp/seq
```

```sql
[zk: localhost:2181(CONNECTED) 13] create -s /temp
Created /temp0000000004
[zk: localhost:2181(CONNECTED) 14] create -s /temp
Created /temp0000000005
[zk: localhost:2181(CONNECTED) 15] create -s /temp
Created /temp0000000006
```

#### EPHEMERAL（临时节点）

- 只存在于当前会话，当对话断开后会被删除。创建的时候增加 -e 

```sql
create -e /temp/seq
```

- 适用于心跳，服务发现等场景。如下图当node1挂掉了，它zookeeper1中创建的临时节点和其存储的数据就会就会被删除，zookeeper1就知道node1挂掉了。

<img src="/Users/panyurou/Library/Application Support/typora-user-images/image-20220116230200083.png" alt="image-20220116230200083" style="zoom:50%;" />

####  EPHEMERAL_SEQUENTIAL（临时序号节点）

与持久序号节点类似，不同之处在于EPHEMERAL_SEQUENTIAL是临时的会在会话断开后删除。创建时添加 -e -s 

```sql
create -e -s /temp/seq
```



