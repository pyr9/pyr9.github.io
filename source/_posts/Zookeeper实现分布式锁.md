---
title: Zookeeper实现分布式锁
date: 2022-09-20 15:05:30
tags:
- 分布式锁
categories: zookeeper
---

# 1基于ZooKeeper的分布式锁。

适用于高可靠（高可用），而并发量不是太高的场景

优点：不存在锁失效的问题，不需要续命。

缺点：**因为需要频繁的创建和删除节点，性能上不如Redis。** 

在高性能、高并发的应用场景下，不建议使用ZooKeeper的分布式锁。而由于ZooKeeper 的高可用性，因此在并发量不是太高的应用场景中，还是推荐使用ZooKeeper的分布式锁

**Zookeeper第三方客户端curator中已经实现了基于Zookeeper的分布式锁。**

## 1.1 **基于Zookeeper设计思路一** 

利用Zookeeper创建临时节点来实现分布式锁，同一路径下的节点名称不能重复，来行防重

- 同一时刻只能允许一个竞争者获取锁。加锁时我们创建一个临时节点。
- 当竞争者A加锁成功后，第竞争者B再来加锁就会抛出节点名称不能重复错误，如果抛出这个异常，我们就判定竞争者B加锁失败
- 竞争者B加锁失败后，会阻塞等待，监听节点状态，当节点数据删除后，也就是竞争者A释放锁，再去获取锁

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h6bzunpe6uj20u0119tap.jpg" style="zoom:50%;" />

## 1.1.1 代码示例

```java
public class ZkDistributedLock extends AbstractLock implements IZkDataListener{

    private ZkClient zkClient ;
    private String path = "/lock";

    private CountDownLatch countDownLatch ;
    private String config;


    @Override
    public boolean tryLock() {
        try {
            if(zkClient ==null){
                //建立连接
                zkClient = new ZkClient(config);
            }
            // 创建临时节点
            zkClient.createEphemeral(path);

        }catch (Exception e){
            //存在节点
            return false;
        }
        return true;
    }



    @Override
    public void waitLock() {
        //注册监听
        zkClient.subscribeDataChanges(path,this);
        if(zkClient.exists(path)){
            countDownLatch = new CountDownLatch(1);
            try {
                countDownLatch.await();  //计数器变为0之前，都会阻塞
                // 解除监听
                zkClient.unsubscribeDataChanges(path,this);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }

    @Override
    public void unlock() {

        if(zkClient !=null){
            zkClient.delete(path);
            System.out.println("-----释放锁资源----");
        }

    }
    
    @Override
    public void handleDataDeleted(String dataPath) throws Exception {
        countDownLatch.countDown();
    }

    @Override
    public void handleDataChange(String dataPath, Object data) throws Exception {

    }

}
```

## 2 基于Zookeeper设计思路二

- 利用Zookeeper创建临时有序节点来实现分布式锁，谁创建的节点序号最小，谁就获得了锁，

- 其他节点就会监听序号比自己小的节点，一旦序号比自己小的节点被删除了，其他节点就会得到相应的事件，然后查看自己是否为序号最小的节点，如果是，则获取锁

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h6c1pzbcvaj20pi0xqmzf.jpg" style="zoom:50%;" />

## 2.1 代码示例

```java
public class ZkDistributedLock2 extends AbstractLock implements IZkDataListener {

    private ZkClient zkClient;
    private String path;
    private String config;

    private CountDownLatch countDownLatch ;
    private String beforePath; // 当前节点前一个节点
    private String currentPath; // 当前节点


    public ZkDistributedLock2(String config, String path){
        this.path = path;
        this.config = config;
        zkClient = new ZkClient(config);
        if (!zkClient.exists(path)) {
            // 如果根节点不存在，则创建根节点
            zkClient.createPersistent(path);
        }
    }

    @Override
    public void lock(){
        //尝试获取锁
        boolean locked = tryLock();
        if(locked){
            System.out.println("---------获取锁---------");
        }
        while (!locked){
            //等待锁 阻塞
            waitLock();
            //重试
            //获取等待的子节点列表
            List<String> childrens = zkClient.getChildren(path);
            //判断，是否加锁成功
            if (checkLocked(childrens)) {
                locked = true;
            }
        }

    }



    @Override
    public boolean tryLock() {
        try {
            //创建临时有序的节点  -e -s
            currentPath = zkClient.createEphemeralSequential(path+"/",null);
            //获取到所有子节点
            List<String> childrens = zkClient.getChildren(path);
            //获取等待的子节点列表，判断自己是否第一个
            if (checkLocked(childrens)) {
                return true;
            }
            // 若不是第一个，则找到自己的前一个节点
            int index =  Collections.binarySearch(childrens,
                    currentPath.substring(currentPath.lastIndexOf("/") + 1));
            if(index<0){
                throw new Exception(currentPath+"节点没有找到" );
            }
            //如果自己没有获得锁，则要监听前一个节点
            beforePath = path + "/" + childrens.get(index-1);
        }catch (Exception e){
            e.printStackTrace();
        }
        return false;
    }


    private boolean checkLocked(List<String> childrens) {

        //节点按照编号，升序排列
        Collections.sort(childrens);

        // 如果是第一个，代表自己已经获得了锁
        if (currentPath.equals(path + "/" +childrens.get(0))) {
            System.out.println("成功的获取分布式锁,节点为"+ currentPath);
            return true;
        }
        return false;
    }

    @Override
    public void waitLock() {
        try {
            countDownLatch = new CountDownLatch(1);
            if(zkClient.exists(beforePath)) {
                //订阅比自己次小顺序节点的删除事件   index-1
                zkClient.subscribeDataChanges(beforePath, this);
                countDownLatch.await();
                zkClient.unsubscribeDataChanges(beforePath, this);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }


    @Override
    public void unlock() {

        if(zkClient !=null){
            //删除临时节点
            zkClient.delete(currentPath,-1);
            System.out.println(currentPath+" 节点释放锁资源");
        }
    }


    @Override
    public void handleDataChange(String dataPath, Object data) throws Exception {

    }

    @Override
    public void handleDataDeleted(String dataPath) throws Exception {
        countDownLatch.countDown(); //减1
    }
}
```

