---
title: zookeeper常见面试题
date: 2023-05-29 23:00:29
tags:
categories:
---

# 1. Zookeeper集群中有几个角色

Zookeeper集群中有三个主要角色：

- **Leader**：负责处理写请求和同步数据到Followers。
- **Follower**：处理读请求，参与选举过程。
- **Observer**：不参与投票选举，仅同步Leader的数据，用于扩展读取性能。

# 2.  Zookeeper如何保证数据一致性？

Zookeeper使用一种称为ZAB协议（Zookeeper Atomic Broadcast）的协议来保证数据的一致性。它确保所有写操作在Leader节点上进行，Leader通过广播将数据同步到Followers，从而确保数据一致性。

# 3. 什么是ZAB协议？

ZAB协议（Zookeeper Atomic Broadcast）是Zookeeper用于保证数据一致性的核心协议。它包括两种主要模式：

- **消息广播模式**：用于正常操作期间，将Leader上的数据更新广播到所有Followers。
- **崩溃恢复模式**：用于Leader崩溃时，选举新的Leader并同步数据。

# 4. Zookeeper集群需要多少个节点来保证高可用性？

Zookeeper集群通常由2N+1个节点组成，以保证在N个节点故障的情况下，集群仍能正常工作。例如，一个由5个节点组成的集群可以容忍2个节点故障。

# 5. 在什么情况下需要进行Leader选举？

- Zookeeper集群启动时，没有Leader。
- 当前Leader节点崩溃或失联。
- 网络分区导致部分节点无法与Leader通信。

# 6. Zookeeper的选举机制是什么？

Zookeeper使用ZAB协议的Leader选举机制来选举新的Leader。选举机制主要分为以下几种情况：

- **启动期间的Leader选举**：当Zookeeper集群启动时，没有Leader，所有节点会进行投票选举Leader。
- **Leader崩溃后的Leader选举**：如果Leader节点崩溃，剩余节点会重新选举新的Leader。

# 7. 选举过程是怎样的？

1. **启动选举**：当ZooKeeper集群启动时，所有节点（Server）都会处于 LOOKING 状态，表示它们在寻找一个领导者。

2. **投票阶段**：

   - 每个节点向其他所有节点发送一条投票信息。投票信息包含节点自身的ID（myid）、其当前的ZXID（事务ID）、以及它认为的Leader的ID和ZXID。

   - 节点在初始状态下会投自己作为Leader，并向其他节点发送这个投票。

3. **接收投票**：

   - 每个节点接收到来自其他节点的投票信息后，会对这些投票进行比较。

   - 比较的依据是首先看ZXID，ZXID大的节点更有可能成为Leader。如果ZXID相同，则比较节点ID，ID大的节点会被选为Leader。

4. **更新投票**：

   - 如果一个节点接收到的投票信息中，存在一个比自己当前投票更高的（根据ZXID和节点ID比较），那么该节点会更新自己的投票信息，并重新发送投票。
   - 这样不断地更新和传播投票信息，最终所有节点都会趋向于认可同一个Leader。

5. **胜出者**：

   - 当某个节点发现自己收到的大多数投票（超过半数）都认可同一个节点作为Leader时，这个节点就会认为选举完成，并将自己状态从LOOKING转换为FOLLOWING或LEADING。
   - Leader会将其他节点的状态从LOOKING转换为FOLLOWING或OBSERVING。

6. **完成选举**：一旦大多数节点达成一致，选举过程结束，Leader节点开始工作，Follower节点开始同步数据。。

![image-20240707232914392](https://panyuro.oss-cn-beijing.aliyuncs.com/image-20240707232914392.png)

# 8. Leader选举的算法是什么？

Zookeeper使用Fast Paxos算法的一种变种进行Leader选举。选举过程通过比较每个节点的epoch（时代编号）和zxid（事务ID）来决定Leader，选择epoch和zxid最大的节点作为Leader。

# 9.  Zookeeper如何解决脑裂问题？

Zookeeper使用多数派机制来应对脑裂问题。在Zookeeper集群中：

- 只有超过半数的节点同意的操作才会被执行（如写操作、Leader选举等）。即使原来的集群分裂成多个小集群，那么拥有超过原集群半数节点的集群至多只有一个，因此至多只会有一个小集群可以成功选举出leader，就避免了脑裂。
- 任何一个子集若未达到超过半数节点，则不能对外提供服务。

具体措施包括：

1. **ZAB协议**：Zookeeper Atomic Broadcast协议确保在Leader崩溃后重新选举新的Leader，并同步数据，保证一致性。
2. **选举机制**：Zookeeper通过快速选举算法选择新的Leader，确保选举过程高效可靠。
3. **同步机制**：新的Leader当选后，会与Followers进行数据同步，确保所有节点数据一致。

### 示例

假设有一个由5个节点组成的Zookeeper集群：

- 在正常情况下，所有5个节点互相通信，协调一致。
- 发生网络分区后，节点可能被分为两个子集，一个包含3个节点，另一个包含2个节点。
- 由于Zookeeper使用多数派机制，包含3个节点的子集可以继续对外提供服务（因为3 > 5/2），而包含2个节点的子集则暂停服务。

通过这些机制，Zookeeper有效避免了脑裂问题，确保系统的一致性和正确性。



**ZooKeeper集群Leader选举中,为 什么必须超过集群总节点数量一半的节点收到相同投票,才能选举出 Leader?**

答:是为了防止因为集群脑裂导致整个ZooKeeper集群不可用。
