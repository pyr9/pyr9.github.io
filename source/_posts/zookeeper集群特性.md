---
title: zookeeper集群特性
date: 2021-09-24 20:41:20
tags: 分布式框架
categories: zookeeper
---

## Zookeeper集群

---

## 目的

zookeeper集群是为了保证系统的性能，能够承载更多的客户端连接。通过集群可以实现以下功能：

* 读写分离：提高承载，为更多的客户端提供连接，并保障性能。
* 主从自动切换：提高服务容错性，部分节点故障不会影响整个服务集群。

##  半数以上运行机制说明

集群至少需要三台服务器，并且强烈建议使用奇数个服务器。

因为zookeeper 通过判断大多数节点的存活来判断整个服务是否可用。比如3个节点，挂掉了2个表示整个集群挂掉，而用偶数4个，挂掉了2个也表示其并不是大部分存活，因此也会挂掉。

## 集群部署

### 配置语法：
```server.<节点ID>=<ip>:<数据同步端口>:<选举端口>```

* 节点ID：服务id手动指定1至125之间的数字，并写到对应服务节点的 {dataDir}/myid 文件中。
* IP地址：节点的远程IP地址，可以相同。但生产环境就不能这么做了，因为在同一台机器就无法达到容错的目的。所以这种称作为伪集群。
* 数据同步端口：主从同时数据复制端口，（做伪集群时端口号不能重复）。
* 远举端口：主从节点选举端口，（做伪集群时端口号不能重复）。

**配置文件示例：**

```
tickTime=2000
dataDir=/var/lib/zookeeper/
clientPort=2181
initLimit=5
syncLimit=2
#以下为集群配置，必须配置在所有节点的zoo.cfg文件中
server.1=zoo1:2888:3888
server.2=zoo2:2888:3888
server.3=zoo3:2888:3888
```

### 集群配置流程：

1. 分别创建3个data目录用于存储各节点数据

```
mkdir data
mkdir data/1
mkdir data/3
mkdir data/3
```

2. 编写myid文件

```
echo 1 > data/1/myid
echo 3 > data/3/myid
echo 2 > data/2/myid
```

```java
ls -R data
1 2 3

data/1:
myid

data/2:
myid

data/3:
myid
```

3、编写配置文件

* `vim conf/zoo1.cfg`

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=data/1
clientPort=2181
#集群配置
server.1=127.0.0.1:2887:3887
server.2=127.0.0.1:2888:3888
server.3=127.0.0.1:2889:3889
```

- `vim conf/zoo2.cfg`

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=data/2
clientPort=2182
#集群配置
server.1=127.0.0.1:2887:3887
server.2=127.0.0.1:2888:3888
server.3=127.0.0.1:2889:3889
```

- `vim conf/zoo3.cfg`

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=data/3
clientPort=2183
#集群配置
server.1=127.0.0.1:2887:3887
server.2=127.0.0.1:2888:3888
server.3=127.0.0.1:2889:3889
```

4.分别启动

```
./bin/zkServer.sh start conf/zoo1.cfg
./bin/zkServer.sh start conf/zoo2.cfg
./bin/zkServer.sh start conf/zoo3.cfg
```

5.分别查看状态

```
./bin/zkServer.sh status conf/zoo1.cfg
Mode: follower
./bin/zkServer.sh status conf/zoo2.cfg
Mode: follower
./bin/zkServer.sh status conf/zoo3.cfg
Mode: leader
```

6. 进入客户端，在server1中创建节点，server 3查看节点是否同步

```sql
./bin/zkCli.sh -server 127.0.0.1:2181,127.0.0.1:2182

[zk: 127.0.0.1:2181,127.0.0.1:2182(CONNECTED) 0] ls /
[zookeeper]
[zk: 127.0.0.1:2181,127.0.0.1:2182(CONNECTED) 1] create -e -s /tt
Created /tt0000000001
[zk: 127.0.0.1:2181,127.0.0.1:2182(CONNECTED) 2] ls /
[tt0000000001, zookeeper]
```

```sql
./bin/zkCli.sh -server 127.0.0.1:2183

[zk: 127.0.0.1:2183(CONNECTED) 1] ls /
[tt0000000001, zookeeper]
```



## 集群角色说明

zookeeper 集群中总共有三种角色，分别是leader（主节点）follower(子节点) observer（次级子节点）

| 角色         | 描述                                                         |
| :----------- | :----------------------------------------------------------- |
| **leader**   | 主节点，又名领导者。用于写入数据，通过选举产生，如果宕机将会选举新的主节点。 |
| **follower** | 子节点，又名追随者。用于实现数据的读取。同时他也是主节点的备选节点，并用拥有投票权。 |
| **observer** | 次级子节点，又名观察者。用于读取数据，与fllower区别在于没有投票权，不能选为主节点。并且在计算集群可用状态时不会将observer计算入内。 |

**observer配置：**
只要在集群配置中加上observer后缀即可，示例如下：

```
server.3=127.0.0.1:2889:3889:observer
```

## 选举机制

通过 ./bin/zkServer.sh status <zoo配置文件> 命令可以查看到节点状态，可以发现中间的2182 是leader状态

```
./bin/zkServer.sh status conf/zoo1.cfg
Mode: follower
./bin/zkServer.sh status conf/zoo2.cfg
Mode: leader
./bin/zkServer.sh status conf/zoo3.cfg
Mode: follower
```

### 投票机制说明

1. 第一轮投票全部投给自己

2. 第二轮投票给myid比自己大的相邻节点，

3. 如果得票超过半数，选举结束。

其选举机制如下图：

![](https://tva1.sinaimg.cn/large/008i3skNly1gywytfrbw1j31bi0u0gpa.jpg)

### **选举触发：**
当集群中的服务器出现已下两种情况时会进行Leader的选举

1. 服务节点初始化启动。当节点初始起动时会在集群中寻找Leader节点，如果找到则与Leader建立连接，其自身状态变化**follower**或**observer。**如果没有找到Leader，当前节点状态将变化LOOKING，进入选举流程。
2. 半数以上的节点无法和Leader建立连接。在集群运行其间如果有follower或observer节点宕机只要不超过半数并不会影响整个集群服务的正常运行。但如果leader宕机，将暂停对外服务，所有follower将进入LOOKING 状态，进入选举流程。

## 数据同步机制

zookeeper 的数据同步是为了保证各节点中数据的一至性，同步时涉及两个流程，一个是正常的客户端数据提交，另一个是集群某个节点宕机在恢复后的数据同步。

### 客户端写入请求

写入请求的大至流程是，收leader接收客户端写请求，并同步给各个子节点。如下图：

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gywz51yujxj30xi0r2taw.jpg" style="zoom:50%;" />

但实际情况要复杂的多，比如client 它并不知道哪个节点是leader 有可能写的请求会发给follower ，由follower在转发给leader进行同步处理

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gywz6vog9pj316g0u00wf.jpg" style="zoom: 50%;" />

客户端写入流程说明：

1. client向zk中的server发送写请求，如果该server不是leader，则会将该写请求转发给leader server，leader将请求事务以proposal（建议）形式分发给follower；
2. 当follower收到收到leader的proposal时，根据接收的先后顺序处理proposal；
3. 当Leader收到follower针对某个proposal过半的ack后，（即follower过半都已经同步完成）则发起事务提交，重新发起一个commit的proposal
4. Follower收到commit的proposal后，记录事务提交，并把数据更新到内存数据库；
5. 当写成功后，反馈给client。

### 服务节点初始化同步
- 在集群运行过程当中如果有一个follower节点宕机，由于宕机节点没过半，集群仍然能正常服务。

- 当leader 收到新的客户端请求，此时无法同步给宕机的节点。造成数据不一致。为了解决这个问题，当节点启动时，第一件事情就是找当前的Leader，比对数据是否一致。不一致则开始同步,同步完成之后在进行对外提供服务。故在节点同步数据期间，该节点不会对外提供服务。

- Leader挂了后，选举leader的过程中，集群不可以对外提供服务。

### 5.四字运维命令

ZooKeeper响应少量命令。每个命令由四个字母组成。可通过telnet或nc向ZooKeeper发出命令。
这些命令默认是关闭的，需要配置4lw.commands.whitelist来打开，可打开部分或全部示例如下：

```
#打开指定命令
4lw.commands.whitelist=stat, ruok, conf, isro
#打开全部
4lw.commands.whitelist=*
#查看服务器及客户端连接状态
echo stat | nc localhost 2181
```

```java

➜  apache-zookeeper-3.7.0-bin echo stat | nc localhost 2181
stat is not executed because it is not in the whitelist.

## 打开全部nc命令
➜  apache-zookeeper-3.7.0-bin echo "4lw.commands.whitelist=*" >> conf/zoo1.cfg
➜  apache-zookeeper-3.7.0-bin echo "4lw.commands.whitelist=*" >> conf/zoo2.cfg
➜  apache-zookeeper-3.7.0-bin echo "4lw.commands.whitelist=*" >> conf/zoo3.cfg

## 重启服务
➜  apache-zookeeper-3.7.0-bin ./bin/zkServer.sh restart conf/zoo1.cfg
➜  apache-zookeeper-3.7.0-bin ./bin/zkServer.sh restart conf/zoo2.cfg
➜  apache-zookeeper-3.7.0-bin ./bin/zkServer.sh restart conf/zoo3.cfg
```

###  ZXID说明

> 如何比对Leader的数据版本呢，这里通过ZXID事物ID来确认。比Leader小就需要同步。

- ZXID是一个长度64位的数字，其中低32位是按照数字递增，任何数据的变更都会导致低32位的数字加1。

- 高32位是leader周期编号，每当选举出一个新的leader时，新的leader就从本地事物日志中取出ZXID,然后解析出高32位的周期编号，进行加1，再将低32位的全部设置为0。这样就保证了每次新选举的leader后，保证了ZXID的唯一性而且是保证递增的。 

eg1: 节点数据的变更

![](https://tva1.sinaimg.cn/large/008i3skNly1gyx6ctcstxj31460siaea.jpg)

Eg2: 某个节点挂掉重新选举

![](https://tva1.sinaimg.cn/large/008i3skNly1gyx6bhhfxpj30we0qmgpn.jpg)

![](https://tva1.sinaimg.cn/large/008i3skNly1gyx66k725nj31620tytd3.jpg)

**思考题：**
如果leader 节点宕机，在恢复后它还能被选为leader吗？

不会，因为它的数据不是最新的。

