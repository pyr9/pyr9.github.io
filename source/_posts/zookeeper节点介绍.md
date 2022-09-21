---
title: zookeeper节点介绍
date: 2022-09-20 15:46:47
tags: 分布式框架
categories: zookeeper
---

# 1 Zookeeper 数据模型

* Zookeeper 数据模型的结构是基于节点的，我们把这种节点叫做 **Znode** 。
* 节点之下可以包含子节点，这种结构和数据结构中的树类似，也和文件系统的目录类似。

- 每个节点拥有唯一的路径path.

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gycegg0aihj30u00v0wgk.jpg" style="zoom: 33%;" />

# 2 节点的结构

* **path**:唯一路径 
* **childNode**：子节点
* **stat**:状态属性:即path对应的值
* **type**:节点类型

# 3 节点类型

| 类型                  | 描述                           |
| :-------------------- | :----------------------------- |
| PERSISTENT            | 持久节点                       |
| PERSISTENT_SEQUENTIAL | 持久序号节点                   |
| EPHEMERAL             | 临时节点(不可在拥有子节点)     |
| EPHEMERAL_SEQUENTIAL  | 临时序号节点(不可在拥有子节点) |

## 3.1 PERSISTENT（持久节点）

- 持久化保存的节点，也是默认创建的

```sql
create /test
```

## 3.2 PERSISTENT_SEQUENTIAL(持久序号节点)

创建时zookeeper 会在路径上加上序号作为后缀。非常适合用于分布式锁、分布式选举等场景。创建时添加 -s 参数即可。

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

## 3.3 EPHEMERAL（临时节点）

- 只存在于当前会话，当对话断开后会被删除。创建的时候增加 -e 

```sql
create -e /temp/seq
```

- 适用于心跳，服务发现等场景。如下图当node1挂掉了，它zookeeper1中创建的临时节点和其存储的数据就会就会被删除，zookeeper1就知道node1挂掉了。

<img src="/Users/panyurou/Library/Application Support/typora-user-images/image-20220116230200083.png" alt="image-20220116230200083" style="zoom: 33%;" />

## 3.4 EPHEMERAL_SEQUENTIAL（临时序号节点）

与持久序号节点类似，不同之处在于EPHEMERAL_SEQUENTIAL是临时的会在会话断开后删除。创建时添加 -e -s 

```sql
create -e -s /temp/seq
```

```sql
[zk: localhost:2181(CONNECTED) 2] create -e -s /temp/c1
Created /temp/c10000000000
[zk: localhost:2181(CONNECTED) 3] create -e -s /temp/c1
Created /temp/c10000000001
[zk: localhost:2181(CONNECTED) 4] create -e -s /temp/c1
Created /temp/c10000000002
```

# 4 节点属性

- 查看节点属性

```sql
stat /temp
```

- 属性说明

```sql
cZxid = 0x2   #创建节点的事务ID
ctime = Mon Jan 17 21:09:16 CST 2022 #创建节点的时间
mZxid = 0x2 #修改当前节点数据的事务ID
mtime = Mon Jan 17 21:09:16 CST 2022 #最后修改时间
pZxid = 0x5 #子节点变更（包括自节点的删除和增加，不包括修改）的事务ID
cversion = 3  #这表示对此znode的子节点进行的更改次数，只包括自节点的删除和增加
dataVersion = 0 #数据版本，当前节点数据变更次数
aclVersion = 0 #权限版本，变更次数
ephemeralOwner = 0x0 #临时节点所属会话ID，永久节点值为0x0
dataLength = 0 #数据长度
numChildren = 3 #子节点数(不包括子子节点)
```

### 

# 5 Znode 节点的操作

## 5.1 **创建节点：** `create`

```bash
# 创建一个持久节点
create /persistent_node
# 创建一个持久的顺序节点
create -s /persistent_sequential_node
# 创建一个临时节点
create -e /ephemeral_node
# 创建一个临时的顺序节点
create -s -e /ephemeral_sequential_node


代码块12345678
```

- 

```java
# 创建一个持久节点
create /persistent_node
# 创建一个持久的顺序节点
create -s /persistent_sequential_node
# 创建一个临时节点
create -e /ephemeral_node
# 创建一个临时的顺序节点
create -s -e /ephemeral_sequential_node
```



## 5.2 **删除节点：** `delete`

```java
delete /config/topics/test
```

## 5.3 **获得一个节点的数据：** `get`

```java
get /persistent_node
```

## 5.4 **设置一个节点的数据：**

```java
set /brokers myNewData
```

## 5.5 **获取子节点：**

```java
# 获取根节点下的子节点
ls /
# 根节点下的子节点有 zk-watcher-2，zookeeper
[zk-watcher-2, zookeeper]
```



# 6 节点的监听

客户添加 -w 参数可实时监听节点与子节点的变化，并且实时收到通知。非常适用保障分布式情况下的数据一致性。其使用方式如下：

| 命令                 | 描述                                 |
| :------------------- | :----------------------------------- |
| ls -w path           | 监听子节点的变化（增，删）           |
| get -w path          | 监听节点数据的变化                   |
| stat -w path         | 监听节点属性的变化                   |
| printwatches on\|off | 触发监听后，是否打印监听事件(默认on) |

## 6.1 监听节点数据的变化

- step1：打开客户端A

  ```sql
  get -w  /temp
  ```

- Step2：打开客户端B:

  ```sql
  set /temp "aaa"
  ```

- 此时客户端A，出现：

  ```sql
  [zk: localhost:2181(CONNECTED) 8] get -w  /temp
  null
  [zk: localhost:2181(CONNECTED) 9]
  WATCHER::
  
  WatchedEvent state:SyncConnected type:NodeDataChanged path:/temp
  ```

- 再次在客户端B输入：

  ```sql
  set /temp "ddd"
  ```

  此时可以发现，客户端A没有收到任何通知，故这里的watch是一次性的，第二次触发不再生效。

## 6.2 监听子节点的变化

- step1：打开客户端A

  ```sql
  ls -w  /temp
  ```

- Step2：打开客户端B:

  ```sql
  create -e -s /temp/c1
  ```

- 此时客户端A，出现：

  ```sql
  [zk: localhost:2181(CONNECTED) 9] ls -w /temp
  [c10000000000, c10000000001, c10000000002]
  [zk: localhost:2181(CONNECTED) 10]
  WATCHER::
  
  WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/temp
  ```


# 7 Zookeeper权限控制

- ACL全称为Access Control List（访问控制列表），用于控制资源的访问权限。   
- ZooKeeper使用ACL来控制对其znode的防问。
- 基于`scheme:id:permission`的方式进行权限控制。scheme表示授权模式、id模式对应值、permission即具体的增删改权限位。

## 7.1 scheme:授权模型

| 方案   | 描述                                                         |
| :----- | :----------------------------------------------------------- |
| world  | 开放模式，world表示全世界都可以访问（这是默认设置）          |
| ip     | ip模式，限定客户端IP防问                                     |
| auth   | 用户密码认证模式，只有在会话中添加了认证才可以防问           |
| digest | 与auth类似，区别在于auth用明文密码，而digest 用sha-1+base64加密后的密码。在实际使用中digest 更常见。 |

## 7.2 **permission权限位**

| 权限位 | 权限   | 描述                             |
| :----- | :----- | :------------------------------- |
| c      | CREATE | 可以创建子节点                   |
| d      | DELETE | 可以删除子节点（仅下一级节点）   |
| r      | READ   | 可以读取节点数据及显示子节点列表 |
| w      | WRITE  | 可以设置节点数据                 |
| a      | ADMIN  | 可以设置节点访问控制列表权限     |

## 7.3 acl 相关命令

| 命令    | 使用方式                | 描述         |
| :------ | :---------------------- | :----------- |
| getAcl  | getAcl <path>           | 读取ACL权限  |
| setAcl  | setAcl <path> <acl>     | 设置ACL权限  |
| addauth | addauth <scheme> <auth> | 添加认证用户 |

#### 示例

- world

```sql
[zk: localhost:2181(CONNECTED) 11] getAcl /temp
'world,'anyone
: cdrwa
[zk: localhost:2181(CONNECTED) 12] setAcl /temp world:anyone:rwa
[zk: localhost:2181(CONNECTED) 13] getAcl /temp
'world,'anyone
: rwa
[zk: localhost:2181(CONNECTED) 14] get /temp
ddd
[zk: localhost:2181(CONNECTED) 15] set /temp "bbb"
[zk: localhost:2181(CONNECTED) 16] create -e /temp/t
Insufficient permission : /temp/t
```

- IP

  语法： setAcl <path> ip:<ip地址|地址段>:<权限位>

```
[zk: localhost:2181(CONNECTED) 18] setAcl /temp ip:192.168.2.103:ra
[zk: localhost:2181(CONNECTED) 19] getAcl /temp
Insufficient permission : /temp
```

- auth

  语法： 

  1. setAcl <path> auth:<用户名>:<密码>:<权限位>
  2. addauth digest <用户名>:<密码>

- digest

  ```sql
  ➜  ~ echo -n pyr:123456 | openssl dgst -binary -sha1 | openssl base64
  zj9LbzdoKijw/kCo1pQJnXBFiq4=
  ```

  ```
  [zk: localhost:2181(CONNECTED) 28] create /temp2
  Created /temp2
  [zk: localhost:2181(CONNECTED) 30] getAcl /temp2
  'world,'anyone
  : cdrwa
  [zk: localhost:2181(CONNECTED) 31] setAcl /temp2 digest:pyr:zj9LbzdoKijw/kCo1pQJnXBFiq4=:rwa
  [zk: localhost:2181(CONNECTED) 32] getAcl /temp2
  Insufficient permission : /temp2
  [zk: 127.0.0.1:2181(CONNECTED) 45] addauth digest pyr:123456
  [zk: 127.0.0.1:2181(CONNECTED) 46] get /temp2
  null
  ```

  

