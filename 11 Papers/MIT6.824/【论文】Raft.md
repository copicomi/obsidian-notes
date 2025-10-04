[[【分布式】raft-extended.pdf]]
# Abstract 
Raft 是一个共识性算法，与paxos等效，但更简单
将leader选举、日志复制、安全性三个模块隔离开

# 2 复制状态机

采用 log 用于崩溃恢复

# 3 Paxos 的缺点
## 太难
## 不容易实现

# 4 Raft 简化的设计
1. 将不同模块分开设计
2. 减少状态机的状态个数

# 5 Raft 算法
三步走：
  1. 选出 leader，用于管理复制log
  2. leader 从 client 接收 log，并进行广播
  3. 当 leader 死亡时，选举新的leader

> 这种由 leader 集中处理 log 的设计，减少了不同节点之间的多余通信

以 leader 设计为基础，Raft 将要解决三部分问题：
  1. 选举 leader 
  2. 复制 log
  3. 维持状态一致性

## 5.1 basics

三种角色：
1. leader
2. follower
3. candidate
![[Pasted image 20250904110310.png|600]]
我们将时间划分为不同的 **term**，用于在逻辑上区分不同的leader区间
每个 term 都遵循 选举->运行 的流程，保证至多有一个 leader

通过维护 termID，server能够得知自己是否落后于系统
  如果是这样，说明新的选举已经完成，自动放弃当前身份成为 follower
![[Pasted image 20250904110713.png|600]]

不同的server之间使用 RPC 进行通信

## 5.2 Leader election
如果一个 leader 存活，它会定期广播**heartbeat**，通知所有人leader仍有效
  当一个server很久没有收到 heartbeat 时，便认定当前 leader 已失效，进入选举阶段

一个 candidate 通过广播requestVote获取选票，获取半数以上的票才能成为leader
  一个 server 只能投出一票，因此采用**先到先投**的原则，只给第一个收到的请求投票
  这种设计保证只能选出唯一的leader

然而，票数不能集中时，没有人会成为leader，进入下一个选举term也不能集中票数
  因此，我们采用**随机化 timeout** 的策略，让每个 server 成为 candidate 的时间随机化
  这样先发起选举的 candidate 就更有优势，票数更容易集中

> 也可以采用预分配 rank 的方法防止选举失败，但这种方法比较低效

选举成功后，新 leader 会广播heartbeat，通知所有人已经选出新的leader

## 5.3 Log replication
leader接收其他server的log，并广播到全网
  如果有server没有成功接收，leader会持续发送log，直至成功
  server本地记录log的term号与index

当一条log被分发到半数以上的server时，认定该log成功提交**commit**
  在此之前的log，都可以被apply到server的状态机

log队列由leader管理，只需保证所有server的log与leader一致即可
  当server收到错序log时，直接拒绝
  多次迭代后，全部server的log会保证一致性

然而，若迭代完成之前leader就已经死亡，此时各个server之间的log是不一致的
  即便选举出新的leader，新log也可能落后或超前
  此时需要先恢复log到一致状态

![[Pasted image 20250904124549.png|600]]

保证 leader 的地位，我们让server去追随 leader 的log
  当新 leader 启动时，会不断检查当前log与每个server的冲突
  使用 **NextIndex** 变量进行**线性探测**，可以大步优化
  探测到公共部分后，将冲突 log 删除，加入 leader 的 log 即可
  完成时就保证了一致性
  
> __我觉得有一个case有疑问：
>   如果一个log被半数以上的server成功commit，修改了状态机
>   而此时偏偏选出了一个落后的leader，状态机是不可逆的
>   这种情况如何解决？__

到这里leader保证了log的一致性，没有考虑状态机

## 5.4 safety
为了解决状态机一致性的问题，我们对选举增加新的限制条件
### 5.4.1 选举限制
前面提到过，一个被commit的log，一定出现在半数以上的server中
  而选举也需要半数以上的server

一个candidate想成为leader，必须经过至少一个新server的同意
  因此我们选择在 RequestVote 中传递 candidate 的 log 信息
  如果 server 发现 candidate 存在**没有 commit 的 log**，就拒绝投票
  那么一个过时的 candidate 一定会收到半数以上的反对票，不会成为 leader

### 5.4.2 前任 leader 的 commit
如果一个leader在commit前死亡
  此时log已经分发到超过半数server，事实上已经commit
  然而 leader 却没有对其进行标记
  导致可能有落后的 server 成为 leader，覆盖掉已经 commit 的 log

如何处理这种遗留的commit呢？
  Raft 的方案是，限制 commit 标准，**只允许当前任期的leader根据计数进行commit**
  对于遗留的 commit，认为属于之前任期的leader，直接进行覆盖

Raft 采用这种保守策略，减少了复杂度

### 5.4.3 正确性证明

分别证明leader广播的log具有持久性，状态机的转移具有一致性即可

## 5.5 follower, candidate crashes
其他 server 死亡时，leader只需要不断重发 RPC，直到有效即可

## 5.6 timing 和可用性
参数限制：$broadcastTime \ll electionTimeout \ll MTBF$
  传播时间 broadcastTime ：0.5ms ~ 20ms
  选举时限 electionTimeout ：10ms ~ 500ms
  故障时间 MTBF：几个月或更长

# 7 log压缩
使用 snapshot 或 LSM-tree 进行压缩
  如果使用 snapshot，需要增加一个 InstallSnapshot RPC 来发送快照
