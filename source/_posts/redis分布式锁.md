---
title: 分布式锁
date: 2021-12-29 23:45:20
tags:
- 分布式框架
- redis
- mysql
- zookeeper
categories: Zookeeper
---

# 1 什么是分布式锁

- 在单体的应用开发场景中涉及并发同步的时候，大家往往采用Synchronized（同步）或者其他同一个JVM内Lock机制来解决多线程间的同步问题。
- 在分布式集群工作的开发场景中，就需要一种更加高级的锁机制来处理跨机器的进程之间的数据同步问题，这种跨机器的锁就是分布式锁

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h6b0wfci7jj21m80mgmzk.jpg)

# 2 分布式锁实现方案

## 2.1 基于数据库实现分布式锁

基于数据库实现分布式锁主要是利用数据库的唯一索引来实现，唯一索引天然具有排他性，这刚好符合我们对锁的要求

### 2.1.1 设计原理

- 同一时刻只能允许一个竞争者获取锁。加锁时我们在数据库中插入一条记录，利用唯一键进行防重。
- 当竞争者A加锁成功后，第竞争者B再来加锁就会抛出唯一索引冲突，如果抛出这个异常，我们就判定竞争者B加锁失败
- 竞争者B加锁失败后，会阻塞等待，一直到竞争者A释放锁（也就是删除记录后），再去获取锁

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h6bwc6rlfpj20vf0u0401.jpg" style="zoom:50%;" />





### 2.2.2 实现注意事项

- 没有锁超时机制。如果程序发生了异常，将无法删除数据，也就是锁无法被释放掉，需要自己写一套锁超时机制，比如：在表中新增一列，用于记录失效时间，并且需要有定时任务清除这些失效的数据；
- 基于数据库实现的，数据库的可用性和性能将直接影响分布式锁的可用性及性能，可以考虑实现数据库的高可用方案
- 自旋实现阻塞效果。当获取锁失败时自旋转

> db操作性能较差，并且有锁表的风险，一般不考虑。

### 2.2.3 代码示例

```java
 */
public abstract class AbstractLock implements Lock{

    /**
     * 加锁，增加重试逻辑
     */
    @Override
    public void lock() {
        //尝试获取锁
        if(tryLock()){
            System.out.println("---------获取锁---------");
        }else {
            //等待锁 阻塞
            waitLock();
            //重试
            lock();
        }
    }
}
```



```java
public class MysqlDistributedLock extends AbstractLock {

    @Autowired
    private MethodlockMapper baseMapper;

    @Override
    public boolean tryLock() {
        try {
            //插入一条数据   insert into
            baseMapper.insert(new Methodlock("lock"));
        }catch (Exception e){
            //插入失败
            return false;
        }
        return true;
    }

    @Override
    public void waitLock() {
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void unlock() {
        //删除数据   delete
        baseMapper.deleteByMethodlock("lock");
        System.out.println("-------释放锁------");
    }

```



## 2 **基于Redis实现分布式锁**

效率最高，加锁速度最快，因为Redis几乎都是纯内存操作

适用于并发量很大、性能要求很高而可靠性问题可以通过其 他方案去弥补的场景。 

## 2.1 设计原理

利用Redis的`SETNX key value`这个命令，只有当key不存在时才会执行成功，如果key已经存在则命令执行失败。

## 2.2.2 实现注意事项

- 需要设置一个超时时间，因为有可能宕机或者被运维重启了，无法释放锁
- 如果业务执行时间> 过期时间，就需要**锁续命**：搞一个定时任务，设一个间隔时间，小于失效时间，过一段时间去监控业务是否执行完了，执行没结束，也就是锁还没释放，我就再把锁的过期时间重新设置成初始值

## redis分布式锁

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h6b11433hoj21kg0u0adf.jpg)

- 多个线程，只有一个线程会执行成功
- 必须使用try catch 在finally中释放锁，否则有异常，锁便没法释放，其他线程进来就会一直执行失败
- 分布式锁需要设置一个超时时间，因为有可能宕机或者被运维重启了，需要保证原子性，写成一个命令

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h6b1621v7yj21ec0goq5r.jpg)

存在问题：现在有两个线程A，B，加锁时长设置的是10秒

- 线程A先去加锁，执行完所有的逻辑需要 15秒，他加锁成功后，正在执行下面的逻辑，第10秒的时候，锁被释放了
- 线程B加锁成功，执行完所有的逻辑需要 8，执行完逻辑后，删除这把锁
- 线程A逻辑执行完了，去释放锁，但锁已经被线程B释放掉了

解决方式，给每一个线程分配一个UUID

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h6b1hwkaarj21jw0scn1d.jpg)

存在问题：

获取client id 和删除redis必须是原子性操作，因为可能存在线程A执行到删除锁之前刚好到失效时间了，然后又卡顿了，这时候线程B就又可以获得锁，释放锁，导致线程A又没法释放锁

解决方式：

锁续命：搞一个定时任务，设一个间隔时间，小于失效时间，过一段时间去监控业务是否执行完了，执行没结束，也就是锁还没释放，我就再把锁的过期时间重新设置成初始值



redison

使用：

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h6b1ygqebkj217s0ccwfs.jpg)

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h6b1z3x7w3j20eq034gll.jpg)

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h6b1wzvpx3j21dn0u0tck.jpg)

原理：

![](https://tva1.sinaimg.cn/large/e6c9d24ely1h6b22djvmej21260o8whq.jpg)



# 3 基于ZooKeeper的分布式锁。

适用于高可靠（高可用），而并发量不是太高的场景

优点：不存在锁失效的问题，不需要续命。

缺点：**因为需要频繁的创建和删除节点，性能上不如Redis。** 

在高性能、高并发的应用场景下，不建议使用ZooKeeper的分布式锁。而由于ZooKeeper 的高可用性，因此在并发量不是太高的应用场景中，还是推荐使用ZooKeeper的分布式锁

**Zookeeper第三方客户端curator中已经实现了基于Zookeeper的分布式锁。**

## 3.1 **基于Zookeeper设计思路一** 

利用Zookeeper创建临时节点来实现分布式锁，同一路径下的节点名称不能重复，来行防重

- 同一时刻只能允许一个竞争者获取锁。加锁时我们创建一个临时节点。
- 当竞争者A加锁成功后，第竞争者B再来加锁就会抛出节点名称不能重复错误，如果抛出这个异常，我们就判定竞争者B加锁失败
- 竞争者B加锁失败后，会阻塞等待，监听节点状态，当节点数据删除后，也就是竞争者A释放锁，再去获取锁

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h6bzunpe6uj20u0119tap.jpg" style="zoom:50%;" />

## 3.1.1 代码示例

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

## 3.2 基于Zookeeper设计思路二

- 利用Zookeeper创建临时有序节点来实现分布式锁，谁创建的节点序号最小，谁就获得了锁，

- 其他节点就会监听序号比自己小的节点，一旦序号比自己小的节点被删除了，其他节点就会得到相应的事件，然后查看自己是否为序号最小的节点，如果是，则获取锁

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h6c1pzbcvaj20pi0xqmzf.jpg" style="zoom:50%;" />

## 3.2.1 代码示例

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

