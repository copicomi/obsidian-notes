# 3A election
ticker 按行为模式划分函数：
1. 发送 heartbeat
2. 发动 election

启动时，运行 `ticker()` 线程，管理定时器相关逻辑

RPC 相关逻辑放在 `rpc.go` 
选举逻辑放在 `election.go`

`RequestVote RPC`
  由 leader 发往 server
  检查投票条件，保证 voteFor 唯一
`AppendEntries RPC`
  由 leader 发往 server
  用于 heartbeat

# 3B Log
需要实现：
1. leader主导的日志分发
2. leader 根据 reply 进行 commit
3. 换任时，检查冲突，同步 leader 的日志
4. 选举限制

## 日志分发
直接发送 append RPC
  在 start() 里调用
## 同步日志
如果有冲突，就减少 nextIndex，重发
## Commit
在 leader 本地进行 commit 更新，同时启动 reply 线程，通知 follower 可以更新状态机
启动 commit 线程 ，定期使用 ticker 检查 commit 进度
## 选举限制
直接比较 log 新旧即可，不要用 commitIndex 比较
先比 term 再比 index

## 设计
- `start()`发送 command，leader接收，并广播 append RPC