本文作为学习备忘录，以个人的commit记录为参考，记录独立完成整个实验的过程，作为简历背书
Lab3 从头写一个 raft 模块，设计细节不再赘述，只记录一些感受到的方法论
作为初学者，一些内容可能比较浅显，见谅

**前置知识**：Raft论文，A Tour of Go

# 3A Election 选举
按照论文实现，注意 ticker 与 处理 VoteRPC 时的细节即可
我想记录的是，在实现 3A 后，我在尽可能写出 *clean code* 方面所做的工作

我认为**下面三个动作**，极大降低了我后续开发的复杂度
1. 拆分代码文件
2. 冗长而有意义的函数命名
3. 设计高阶函数抽象
## 拆分文件
我先前的项目都是在框架代码的基础上做填空，没有组织代码文件的经验，因此这是我个人从零开发的初次实践

选举逻辑就写到 `election.go`，日志处理就写到`log.go`，RPC 通信逻辑就写到 `rpc.go`，等等
我做了这样的划分，明显减轻了开发时切换上下文的心智负担

## 函数命名
我不是一个喜欢写废话注释的人，**最好的注释就是代码本身**，因此我使用了大量封装逻辑

ticker里要发心跳，就单拎出来写一个 `sendHeartbeat()`，同理还有`sendVoteRequest()`
即使整个项目里暂时只有 ticker 会发送 heartbeat，也一定要做好功能封装

这种封装有两个好处：
1. 涉及到 state 转换与判断时，可以使用统一逻辑
2. 通过命名，可以记录 Lock 的使用情况

### 统一逻辑
选举过程中，需要进行 server state 的转换

你当然可以每到一个函数就重新写一遍切换逻辑，
  但再简单的逻辑，如果写上一百次，**难免**会出错一次，那你会付出百倍的代价

即便处理仅仅只有一句简单的 `rf.state = Follower` 或者 `rf.Currenterm += 1`，
  也**绝对**有必要封装成 `rf.TurnIntoFollower()` 或 `rf.IncCurrentTerm()`，
  可读性与可维护性兼备，就该这么写。
  如果你没这么干，那么到开发后期，你新增的逻辑又要复制粘贴到不同地方，**难免**会出错
 
这种封装，同时也减少了你思考细节的成本。
  每次切换到 `Follower`，都要重置 `VoteFor` 等变量，
  这时候只需要在 `TurnIntoFollower()` 中添上几行就完活了，
  再进一步，甚至可以将重置变量的过程，再封装到 `InitFollowerState()` 里

**持续的做封装**，最后你的顶层函数将会充斥着封装逻辑，
  配合冗长而有意义的函数命名，函数名本身就是自然语言，代码本身就是注释

### Lock命名
这是一个分布式项目，意味着**并发**场景与lock，也就意味着死锁风险

加锁还是不加？你总需要思考潜在的死锁问题，如果你**一眼**就能看出调用的函数是否持有锁就好了。
  很明显，`IncCurrentTermWithoutLock()` 这个函数不带锁，从命名入手就解决了

但太多后缀也不好，不可能给每个人都挂上命名牌，我的选择是只对状态处理的函数做这种设计，
  这样的零碎函数，放在`states.go`里面

对于更上层的逻辑，需要从设计层面形成抽象层，根据扮演的 role 判断是否需要锁，我没有考虑过这方面

## 高阶抽象
明显可以注意到，RPC 通信逻辑是一个**与 Raft 整体设计无关**，却又**不得不**在代码里重复的模块

但是每次接收的参数类型又不同，一会发`Append RPC`，一会又发`Vote RPC`，一会广播了，一会又针对某个server复制log了，
  这怎么可能封装成一个函数呢？

针对每个类型都做一层封装逻辑是可以的，但本质上**并没有**减少代码复杂度，仅仅是提高了可读性

```
BoardcastHeartbeat() {
  for i := peers 
    sendAppendRPC(i, MakeEmptyAppendArgs())
}
```
像这样的函数，仅仅是换一换调用的函数，换一换发送的 args，就是另一个功能了，但还是要重复一样的代码

设计的关键是将整个外层包装的 `for i := peers` 抽象出来，内部行为传参调用即可

因此我抽象出 `Boardcast` 广播抽象：
```
Boardcast(sendFunction, handleFunction, argsFactory, replyFactory) {
  for i := rf.peers
    args := argsFactory()
    reply := replyFactory()
    sendFunction(i, args, reply)
    handleFunction(i, args, reply)
}
```

随后对该抽象，进行持续的**复用**即可，你只需要额外写一些工厂函数，放在 `factory.go` 里，
  这样一来，你可以再封装出 `BoardcastHeartbeat`, `BoardcastVoteRequestRPC` 等逻辑，用来反哺可读性

```
BoardcastHeartbeat() {
  args := MakeHeartbeatArgs()
  Boardcast(
    MakeSendFunction(RPCAppendEntries)
    MakeHandleFunction(RPCAppendEntries)
    MakeArgsFactoryWithGivenArgs(RPCAppendEntries, args)
    MakeEmptyReplyFactory(RPCAppendEntries)
  )
}
```

即便调用的类型不同，但我们在高阶里定义的参数都是`interface{}`，
  只要都实现了一样的接口即可复用，通过 `factory MakeFunction` 作为**中间层**进行类型适配

这一part**函数式编程**的思想，是读 SICP 学到的

# 3B Log 同步
依旧按照论文实现，更多的拆分，更多的封装，更多的抽象
3B 里，最大的问题是**并发**情景，*Go routine* 的强大之处在此体现得淋漓尽致

我认为有**两个设计**，减轻了心智负担
1. 后台 task 线程
2. replicator 隔离
## 后台线程
`ticker`,`applier`,`replicator`分别负责不同的业务逻辑，自然可以在后台独立运行
这里我使用了 **sleep** 与 **cond** 两种控制方法

定时器 ticker 用 sleep
而 applier 与 replicator 是事件触发的，监视对应的 `commitIndex` 与 `nextIndex` ，因此用条件变量

而 RPC 通信这样等待 reply 的阻塞行为，也需要用 `go SendAndHandleRPC()` 转到后台运行
## 线程隔离
我对每个server都单独运行一个`replicator`线程，同时保证每个 replicator 都是串行化的，
这样不同 server 之间的同步控制，是彼此**隔离**的

设计之初，我用的是 heartbeat 的回复机制，来判断是否要重发 appendRPC
然而，这么做杂糅了 RPC 处理与 replicate 机制，导致测试过程中频繁出现并发问题

因此我选择将 Retry Send 逻辑封装到 replicator 里，处理 RPC 时只需唤醒 cond 即可

## 重用抽象
即便设计了 Boardcast 抽象，但是 replicator 需要针对单一 server 发送特定的 AppendRPC，
这是不能广播相同 RPC 的情景，不能复用之前的 Boardcast 函数

但也只需要多增加一个参数，做成`BoardcastWithGivenPeers()` 即可，
或者针对 `BoardcastAppendRPC()` 做单独适配也可以

之前提到，上层业务调用的函数尽量要封装好，这里就派上用场了，
尽管改变了底层的逻辑，但上层调用的都是 `BoardcastHeartbeat()` 这样意义明确的封装函数，
只需要略微改动中间层函数的内容即可，也利于单元测试和Debug

不过由于之前提到的并发问题，我最后放弃了对 BoardcastAppend 的重用，转为使用 replicator 线程控制行为

## 更多封装
### 谓词函数
处理 RPC reply 时，要处理很多的边界case，因此需要进行很多 **if 判断**，
我选择将这样的判断封装为*谓词*，命名为`IsMatchPrevLog()`，这样以`Is...()`为格式的函数显式标注谓词，
这样做进一步提高可读性
### Log 处理
我设计了`AppendLogList()`，`AppendSingleLog()`，`CutLogListAtIndex()`等封装，将日志相关状态的变更绑定在一起
在之后实现 Snapshot 的过程中，需要进行 LogIndex 的转换，这时只需修改现在的封装函数即可

# 3C Persist 持久化
这一节本身没有难度，难在更加严格的测试
3C 的主题是 Debug，由于之前的**基础设施**比较完善，我没有花太多时间
## 日志打印
```
func mDebug(rf *Raft, format string, a ...interface{}) {
	if Debug {
		prefix := fmt.Sprintf("[%d] S%d ", rf.currentTerm, rf.me)
		format = prefix + format
		log.Printf(format, a...)
	}
}
```
我没有对日志进行额外处理，只用了上面的 `mDebug()` 打印日志，多测脚本是通义灵码写的
日志能够清晰的指明行为的开始与结束，以及当前的状态即可，方便定位冲突的地方

## 不可靠的网络
框架提供的 `call()` 已经保证通信处理，不需要进行丢包判断，
  这里只需要保证正确处理过期的 RPC args 与 reply

过时的直接拒绝即可，我想强调的是延迟乱序到达的情况

leader先后发送两个 args，但是reply却是以相反的顺序收到，
  这种情况下，RPC处理的状态必须具有**不可逆性**，
  总是应该检查 RPC 是否是当前状态的子集，否则有状态回退的风险

## 性能优化
快速回退，可以利用server方的reply信息进行优化

我的 replicator 采用纯 cond 循环进行控制，但发送了过多的 RPC，
  应该是发送了大量短而频繁的复制请求，因此我增加了一个小小的延时`sleep(10)`，
  这样可以等待 log 堆积，进行批处理

我的 applier 采用 sleep 进行控制，改为 cond 加 sleep 会更优，不过我没有动，
通过 3000/3000 测试后不敢乱动代码了

# 3D Snapshot 快照
这节是最坑的，如果你没有做好封装，这将会是伤筋动骨的一个feat，
依旧按照论文实现，代码写到 `snapshot.go`

## 边界条件
首先我需要加入一个 `snapshotEndIndex` 变量，同时提供截断 `log` 的操作，
因此实现若干个类似`GetLastLogIndexWithoutLock()`的接口即可，在外部依旧使用逻辑index

同时 replicator 根据 `IsExistInSnapshot(nextIndex)` 判断发送 appendRPC 还是 installRPC

这里的 bug 是最晕的，需要从头到尾好好敲打一遍，尤其注意快速回退与replicator的交互，
很多数组越界，非常多

## 死锁
在课程框架的 `server.go` 中 `applierSnap()` 嵌套调用了 `rf.Snapshot()`，
因此在把 message 发送到 applyCh 时会有死锁问题，
起初我仅仅是在 Snapshot 里做了不加锁的处理，这样是不行的，有并发问题

最后我的设计是，单独设计一个小锁，保护 `InstallSnapshot`，`Snapshot` 与 `applier`，
而发送message到applyCh时，临时释放大锁，
snapshot与install有race，因此我用小锁针对这几个函数做互斥处理

## 框架
我觉得课程框架里`server.go`的设计有问题，不应该在 applier 里嵌套调用snapshot，这两个相互独立的逻辑就应该好好隔离开。
同时也存在一个并发 bug，当测试环境模拟 crash 时，将 raft 置为 nil，但没有及时清空信道，仍有可能调用 rf.Snapshot()，导致引用空指针