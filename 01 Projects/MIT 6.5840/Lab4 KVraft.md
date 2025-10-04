# RSM
作为 server 层抽象，提供 submit 接口，通过 DoOp 进行 apply

负责与 raft 层通信，调用 start 后等待 apply 完成

四种信号：
1. apply完成：ok
2. 超时：timeout
3. 非leader：notLeader
4. 停机：killed

其中timeout，可能由于partition引起，无法达成共识，此时上层应该退出，去找另一个leader

不过 timeout 应该在 rsm 层处理吗？

上层串行调用 submit 并等待返回，因此必须提醒超时，不然死循环
而底层 raft 可能会偷偷apply，但上层已经得知 ErrMaybe，会考虑这种情况

不过从测试来看，在分区恢复后，应该正常submit

其实submit的返回结果不重要，如果超时，server会忽略掉submit，而apply也被errMaybe考虑
因此可以将超时判断下放到submit

# server

处理重复op

三层过滤
1. rpc：判断请求是否过期
2. rsm：判断
3. store：请求是否apply

server收到新的rpc时，判断client已收到回复，清理map
server收到旧的rpc时，依旧执行submit，去重由store负责

store收到新的rpc时，调用底层接口，更新状态机
store收到旧的rpc时，仍返回历史记录![[Pasted image 20250919230340.png]]
![[Pasted image 20250919230507.png]]