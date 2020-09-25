# kafka

## page cache
* 为了提高磁盘存取效率,Linux采取了两种主要Cache方式：Buffer Cache和Page Cache。前者针对磁盘块的读写，后者针对文件 inode 的读写。这些Cache有效缩短了 I/O系统调用（比如read,write,getdents）的时间。
* linux下文件close成功，不会触发操作系统刷盘
  
**一次正常的写流程**

一次写数据的典型流程（不考虑异常和其它特殊情况）：

1、数据在用户态的 buffer 中，调用 write 将数据传给内核；

2、数据在 Page Cache 中，返回写入的字节数（成功返回）；

3、内核将数据刷新到磁盘。

第二步如果返回成功，说明数据已经到达操作系统的Page Cache，可以保证的是**如果进程挂了，但是操作系统没挂，数据不会丢失。**

如果调用 fsync 将数据刷新到磁盘上，返回成功，说明数据已经刷新到硬件上了——我们一般认为如果 fsync 返回成功，则表示数据持久化成功

Page Cache 的异步刷新
那么，如果不调用fsync或其它类似功能的接口，Page Cache 是什么时候刷回磁盘的呢？

简单总结一下，有两种情况：

1、脏页太多。

2、脏页太久。

这些都由 Linux 内核的后台线程执行。相关的控制参数有：

```
$ sysctl -a | grep dirty
vm.dirty_background_bytes = 0
vm.dirty_background_ratio = 5
vm.dirty_bytes = 0
vm.dirty_expire_centisecs = 3000
vm.dirty_ratio = 10
vm.dirty_writeback_centisecs = 500
```
说明： centy 中文意思是“百分之十”。


* dirty_writeback_centisecs 表示多久唤醒一次刷新脏页的后台线程。这里的500表示５秒唤醒一次。
* dirty_expire_centisecs 表示脏数据多久会被刷新到磁盘上。这里的3000表示 30秒。
* dirty_background_ratio 表示当脏页占总内存的的百分比超过这个值时，后台线程开始刷新脏页。这个值如果设置得太小，可能不能很好地利用内存加速文件操作。如果设置得太大，则会周期性地出现一个写 IO 的峰值。
* dirty_ratio 当脏页占用的内存百分比超过此值时，内核会阻塞掉写操作，并开始刷新脏页。
* dirty_background_bytes、dirty_bytes 是和 dirty_background_ratio、dirty_ratio 表示同样意义的不同单位的表示。两者不会同时生效。

**总结**
触发刷新脏页的条件：

1、调用fsync等。

2、脏页太多（相关参数：dirty_background_ratio与dirty_ratio）。

3、脏页太久（相关参数：dirty_expire_centisecs）。

## 性能

**kafka尽可能使用磁盘缓存，将数据直接写到操作系统的page cache，而不是jvm里面，保证了可靠性**

* kafka没有使用B树，操作复杂度有O(logN)，对于磁盘操作还是太慢。
影响性能的方面：
* 大量小的磁盘IO和网络IO
  * 解决方案：将信息组合起来一起网络发送，一起写日志
  * 磁盘顺序写性能高
* 大量的数据复制
  * 解决方案：producer broker comsumer共享同一份消息格式，减少转换成本
  * broker相当于文件目录
  * 将块文件发送到网络，可以实现零拷贝  
  * sendfile 操作系统将磁盘数据读到page cache，然后从page cache发送到网络
* 数据冗余
  * 解决方案：端到端的批量压缩
  * 消息存在冗余的内容，比如json的key，web请求的user agent等
  * producer批量压缩一批数据，发给broker，broker直接保存，然后发给consumer，由consumer解压


Q&A？
* broker集群的leader做什么事？
* 批量压缩，偏移量怎么计算
* producer 可以连接任意节点？
* leader是对分区来说的？
* 崩溃节点如何同步数据然后加入集群？
* 节点里的日志顺序如何识别？
* 

## 生产者负载均衡
* 所有broker节点都能提供元数据，告知哪些服务器存活，某个topic的leader是谁
* producer自己决定将数据发给哪个分区，可以随机，可以根据key做hash

## 消费者获取数据
* 消费请求带上需要的偏移量
* 一个分区对应一个消费者，能保证顺序读
* producer push消息，consumer pull 消息
  * push给consumer的缺点
    * 不同的consumer的消费速度不一致，难以确定push的速度
    * 跟不上的consumer会过载
    * 有数据就要推送，缺少下游回馈信息
  * consumer pull 的优点
    * consumer可以提供想要拉取的偏移量，实现批量拉取

## 消息分发元语

* 0.11.0.0版本开始，producer支持幂等发送，producer有ID，每个消息也有个序列号
* 0.11.0.0版本开始，producer支持向多个topic同时发送消息的事务元语，要么都成功，要么都失败
* kafka streams 支持exactly-once 语义，内部使用kafka存储偏移量，使用事务元语
* 配合外部系统，consumer可以实现exactly-once语义，处理结果和偏移量一起事务存储

## 备份
* 备份最小单元是topic分区，每个分区有一个leader和0个或多个followers
* 分区leader处理读写请求
  * 每个Partition有一个leader与多个follower，producer往某个Partition中写入数据是，只会往leader中写入数据，然后数据才会被复制进其他的Replica中。
* 数据是由leader push过去还是有follower pull过来？
  * kafka是由follower周期性或者尝试去pull(拉)过来(其实这个过程与consumer消费过程非常相似)，写是都往leader上写，但是读并不是任意flower上读都行，读也只在leader上读，flower只是数据的一个备份，保证leader被挂掉后顶上来，并不往外提供服务。
* kafka node 存活性有两个要求 in sync
  * 和ZooKeeper保持心跳
  * follower备份数据离leader没有太多
* 选主 要挑有最新备份的follower
  * 多数投票（ majority vote）的Quorum方案
    * 缺点是占据了更多的存储空间，网络传输的数据量也减少了，适合小数据量的集群，不适合大数据存储的集群
    * 优点是备份时间取决于最快的一批节点
  * Kafka动态维护一个同步follower列表（ in-sync replicas (ISR)），这个列表里的follower可以竞争leader
  * 所有的ISR备份了写，才算写成功 commited，时间取决于最慢的节点 
  * Kafka只保证至少有一个备份
* 水印备份机制

## ISR
kafka不是完全同步，也不是完全异步，是一种ISR机制：
1. leader会维护一个与其基本保持同步的Replica列表，该列表称为ISR(in-sync Replica)，每个Partition都会有一个ISR，而且是由leader动态维护
2. 如果一个flower比一个leader落后太多，或者超过一定时间未发起数据复制请求，则leader将其重ISR中移除
3. 当ISR中所有Replica都向Leader发送ACK时，leader才commit

server配置

```
rerplica.lag.time.max.ms=10000
  # 如果leader发现flower超过10秒没有向它发起fech请求，那么leader考虑这个flower是不是程序出了点问题
  # 或者资源紧张调度不过来，它太慢了，不希望它拖慢后面的进度，就把它从ISR中移除。

  rerplica.lag.max.messages=4000 # 相差4000条就移除
  # flower慢的时候，保证高可用性，同时满足这两个条件后又加入ISR中，
  # 在可用性与一致性做了动态平衡   亮点
```

topic配置

```
default.replication.factor 每个分区的备份数，request.required.asks=-1时，当ISR都收到消息了就回复commit成功，这是ISR外的不一定都备份好了
min.insync.replicas=1 # 需要保证ISR中至少有多少个replica
```

Producer配置

```
request.required.asks=0
  # 0:相当于异步的，不需要leader给予回复，producer立即返回，发送就是成功,
      那么发送消息网络超时或broker crash(1.Partition的Leader还没有commit消息 2.Leader与Follower数据不同步)，
      既有可能丢失也可能会重发
  # 1：当leader接收到消息之后发送ack，丢会重发，丢的概率很小
  # -1：当所有的follower都同步消息成功后发送ack.  丢失消息可能性比较低
 ```

* 分区会均匀分布在brokers里，每个broker尽可能都是某些分区的leader
* 有一个broker作为controller，管理所有分区的leader选举，跟 zookeeper 交互后“内定”，再通过 RPC 通知具体的主节点，此举能防止 partition 过多，同时选主导致 zk 过载。controller挂了，另一个broker替上

## Log Compaction
根据key保留最新的值，那怎么保证线性读写呢？
* 同一个key的老数据会删除，保留的那个数据的offset不变