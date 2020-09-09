# Multi-Paxos算法
原始的Paxos算法（Basic Paxos）只能对一个值形成决议，决议的形成至少需要两次网络来回，在高并发情况下可能需要更多的网络来回，极端情况下甚至可能形成活锁。如果想连续确定多个值，Basic Paxos搞不定了。因此Basic Paxos几乎只是用来做理论研究，并不直接应用在实际工程中。 

实际应用中几乎都需要连续确定多个值，而且希望能有更高的效率。Multi-Paxos正是为解决此问题而提出。  

Multi-Paxos基于Basic Paxos做了两点改进：
1. 针对每一个要确定的值，运行一次Paxos算法实例（Instance），形成决议。每一个Paxos实例使用唯一的Instance ID标识。
2. 在所有Proposers中选举一个Leader，由Leader唯一地提交Proposal给Acceptors进行表决。这样没有Proposer竞争，解决了活锁问题。在系统中仅有一个Leader进行Value提交的情况下，Prepare阶段就可以跳过，从而将两阶段变为一阶段，提高效率。

multi-paxos的leader选举需要使用basic-paxos，但是basic-paxos存在活锁问题，那么工程中对此有没有什么办法呢？  
活锁问题与Raft的leader选举的选票分裂问题类似，工程上可采用Raft的leader选举的随机化方法，超时后随机等待一段时间后再发起选举