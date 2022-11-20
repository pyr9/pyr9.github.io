---
title: zookeeper客户端使用
date: 2021-09-23 21:51:20
tags: 分布式框架
categories: Zookeeper
---

## 客户端API常规应用

---

zookeeper 提供了java与C两种语言的客户端。我们要学习的就是java客户端。引入最新的maven依赖：

```java
<dependency>
  <groupId>org.apache.zookeeper</groupId>
  <artifactId>zookeeper</artifactId>
  <version>3.6.2</version>
</dependency>
```

## 初始连接

常规的客户端类是 org.apache.zookeeper.ZooKeeper，实例化该类之后将会自动与集群建立连接。

- connectString    连接串，包括ip+端口 ,集群模式下用逗号隔开
- sessionTimeout  会话超时时间，该值不能超过服务端所设置的 minSessionTimeout 和maxSessionTimeout
- watcher 会话监听器，服务端事件将会触该监听

```java
zooKeeper = new ZooKeeper("192.168.2.103:2181", 4000, new Watcher() {
  @Override
  public void process(WatchedEvent watchedEvent) {
    System.out.println("init:"+watchedEvent.getPath());
  }
});
```

## 创建、查看节点

###  创建节点

通过org.apache.zookeeper.ZooKeeper#create()即可创建节点

```java
  public void createData() throws InterruptedException, KeeperException {
    List<ACL> aclList = new ArrayList<>();
    int perm = ZooDefs.Perms.ADMIN | ZooDefs.Perms.READ;
    
    // perms 对应权限位permission
    // id 对应权限模式scheme + id
    ACL acl1 = new ACL(perm, new Id("world","anyone"));
    ACL acl2 = new ACL(perm, new Id("ip","192.168.2.103"));
    ACL acl3 = new ACL(perm, new Id("ip","127.0.0.1"));

    aclList.add(acl1);
    aclList.add(acl2);
    aclList.add(acl3);
    zooKeeper.create("/hello", "hello word!".getBytes(), aclList, CreateMode.PERSISTENT);
  }
```

运行完成后，在zookeeper进行查看

```java
[zk: 127.0.0.1:2181(CONNECTED) 64] getAcl /hello
'world,'anyone
: ra
'ip,'192.168.2.103
: ra
'ip,'127.0.0.1
: ra
[zk: 127.0.0.1:2181(CONNECTED) 65] get /hello
hello word
```

### 查看节点

#### 查看当前节点

通过org.apache.zookeeper.ZooKeeper#getData()即可查看节点

```java
  public void testData() throws InterruptedException, KeeperException {
    final byte[] keeperData = zooKeeper.getData("/pyr", false, null);
    System.out.println("testData:" + new String(keeperData));
  }
```

#### 查看子节点

通过org.apache.zookeeper.ZooKeeper#getChildren()即可获取子节点

```java
  public void testGetChildrenWithoutWatch() throws InterruptedException, KeeperException {
    final List<String> children = zooKeeper.getChildren("/pyr", false);
    children.stream().forEach(System.out::println);
  }
```

## 监听节点

- 在getData() 与getChildren()两个方法中可分别设置监听数据变化和子节点变化。

- 通过设置watch为true，当前事件触发时会调用zookeeper()构建函数中Watcher.process()方法。也可以添加watcher参数来实现自定义监听。一般采用后者。
- 所有的监听都是一次性的，如果要持续监听需要触发后在添加一次监听。

**使用默认监听**

```java
  public void testGetDataWithWatcher() throws InterruptedException, KeeperException {
    final byte[] keeperData = zooKeeper.getData("/pyr", true, null);
    System.out.println("testGetDataWithWatcher:" + new String(keeperData));
    Thread.sleep(Long.MAX_VALUE);
  }
```

**自定义监听**

```java
  public void testGetDataWithCustomWatcher() throws InterruptedException, KeeperException {
    final byte[] keeperData = zooKeeper.getData("/pyr", new Watcher() {
      @Override
      public void process(WatchedEvent watchedEvent) {
        try {
          zooKeeper.getData("/temp", this, null);
        } catch (KeeperException | InterruptedException e) {
          e.printStackTrace();
        }
        System.out.println("testGetDataWithCustomWatcher:" + watchedEvent.getPath());

      }
    }, null);
    Thread.sleep(Long.MAX_VALUE);
  }
```



## 第三方客户端ZkClient

zkClient 是在zookeeper客户端基础之上封装的，使用上更加友好。主要变化如下：

* 可以设置持久监听，或删除某个监听
* 可以插入JAVA对象，自动进行序列化和反序列化
* 简化了基本的增删改查操作。
