# zookeeper
任何在设计阶段考虑到的异常情况，一定会在系统实际运行中发送，并且，在系统实际运行过程中还会遇到很多在设计时未能考虑到的异常故障。

# 原理

## ACID
* 原子性（Atomicity）
* 一致性（Consistency）
* 隔离项（Isolation）
* 持久性（Durability）


## CAP
一个分布式系统不可能同时满足
* 一致性(C:Consistency)
* 可用性（A:Availability）
* 分区容错性（P:Partition tolerance）
这三个基本需求，最多只能同时满足其中的两项

对一个分布式系统而言，分区容错性是一个基本要求。一般是根据业务特性在C和A直接寻求平衡

## BASE
* 基本可用（basically available)
* 软状态（soft state）
* 最终一致性（eventually consistent）

ACID属于强一致性模型，BASE通过牺牲强一致性来获得可用性，并允许数据在一段时间内是不一致的，但最终达到一致状态。

# 一致性协议

## 2PC 二阶段提交
协调者（Coordinator）
参与者（Participant）
二阶段提交将一个事务的处理方式分为了投票和执行两个阶段，可以看做一个强一致性算法
优点
* 原理简单，实现方便

确定
* 同步阻塞
* 单点问题 协调者是单点
* 数据不一致（脑裂） commit请求未送达所有参与者
* 太过保守 协调组仅能依靠超时机制判断是否需要终端事务，没有较为完善的容错机制，任意一个节点的失败会导致整个事务的失败

## 3PC 三阶段提交
1. CanCommit
2. PreCommit
3. do Commit


一个分布式算法有两个最重要的属性：安全性（Safety）和活性（Liveness）
Safety是指那些需要保证永远不会发送的事情。
Liveness是指那些最终一定会发生的事情。

## Chubby
中心化部署的分布式所服务，而非仅仅是一个一致性协议的客户端库

客户端成功获取到一个Chubby文件锁，成为Master，向文件里写入Master信息，其他客户端通过读取这个文件得知当前的Master信息

Master租期，租约形式：通过不断续租来延长Master租期

虚拟时间 虚拟同步


Master失效检测
Master选举
Master重新选举对客户端的平滑过渡
客户端缓存一致性

从节点恢复后数据恢复，快照+回放


## zookeeper
ZooKeeper为分布式应用提供了高效且可靠的分布式协调服务，提供了诸如统一命名服务、配置管理和分布式锁等分布式的基础服务

ZooKeeper是一个典型的分布式数据一致性的解决方案，分布式应用程序可以基于它实现诸如
* 数据发布/订阅
* 负载均衡
* 命名服务
* 分布式协调/通知
* 集群管理
* Master选举
* 分布式锁
* 分布式队列

适用于以读为主的应用场景

ZooKeeper可以保证如下分布式一致性：
* 顺序一致性 从同一个客户端发起的事务请求，最终会严格的按照发起顺序被应用
* 原子性
* 单一视图 无论客户端连接的是哪个ZooKeeper服务器，其看到的服务端数据模型是一致的
* 可靠性
* 实时性 ZooKeeper仅能保证在一定时间内，客户端最终一定能够从服务端上读取到最新的数据状态
  
### 集群角色
* Leader
* Follower
* Observer

Leader选举过程选定一台机器成为Leader，为客户端提供读和写服务
Follower和Observer提供读服务
Observer不参与Leader选举过程，也不参与写操作的“过半写成功”策略，可以在不影响写性能的情况下提升集群的读性能

## ZAB协议
ZooKeeper Atomic Broadcast Zooke原子消息广播协议
支持崩溃恢复的原子广播协议

包括两种基本的模式：
* 崩溃恢复
* 消息广播

仅允许一个Leader服务器进行事务请求的处理  
Leader服务器在接收到客户端的事务请求后，会生成对应的事务提案并发起一轮广播协议  
如果集群中的其他机器接收到客户端的事务请求，那么这些非Leader服务器会首先将这个事务请求转发给Leader服务器

ZAB类似二阶段提交，在ZAB协议的二阶段提交过程中，移出来终端逻辑，过半的Follower服务器已经反馈ACK之后就开始提交事务Proposal，不需要等待集群中所有的Follower服务器都反馈响应


Leader选举算法：  
能够确保提交已经被Leader提交的事务Proposal，同时丢弃已经被调过的事务Proposal  
保证选举出来的Leader服务器拥有集群中所有机器最高编号（即ZXID最大）的事务Proposal

ZXID 是一个64位的数字，低32位可以看做是一个简单的单调递增计数器，高32位代表Leader周期epoch的编号






