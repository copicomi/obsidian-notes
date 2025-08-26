[[【分布式】MapReduce.pdf]]
# Abstract
MapReduce是一种编程model，用于在分布式集群上执行大规模的并行计算，Map用于生成中间结果，Reduce用于合并结果
> Map与Reduce的灵感来源于 lisp

# 1 Introduction
MapReduce的目的是向用户隐藏分布式架构的底层细节，提供可用的API

# 2 Programming Model
接受与输出 KVpair，计算过程由用户定义，很多事情都可以用Map-Reduce抽象

# 3 Implementation
## 3.1 运行流程
1. MapReduce框架对文件进行分片
2. 选定一个master线程，加载用户传入的procedure
3. map worker 从分片中读数据，传入到 Map 过程，结果输出到缓冲区
4. reduce worker 从缓冲区读数据，传入到 Reduce 过程，结果输出到文件
5. MapReduce框架返回结果
![[Pasted image 20250819164123.png|800]]
## 3.2 Master Data Structure
Master 维护两层 worker 间的数据流，存储以下信息：
1. worker的状态
2. 缓冲区状态

## 3.3 纠错能力
### Worker Failure
该框架对 worker 的容错很高，定期 ping worker，回复异常就认为该 worker 故障
弃用故障的worker，重做对应的task，并改变数据流向即可
### Master
Master一旦出错可以回溯到最近的checkpoint
不过一般情况下会直接abort，让用户决定处理
### Semantics
该系统在分布式实现的情况下，内部语义依旧与顺序执行相同
MapReduce使用自动commit记录，解决并发冲突

## 3.4 工作环境
我们倾向于让 worker 在多台机器上分散运行，传递计算结果而不是原数据

## 3.5 参数
可以调整分片个数 M 与输出文件数 R

## 3.6 Backup Tasks
Worker的执行速度不平衡，可能有一个极慢的worker增加整体延迟
解决方案是，在整体流程快结束时，为过慢的task创建备份，在闲置的worker上并行计算

# Conclusion
该模型的优势
1. 隐藏分布式底层的复杂度
2. 实现合理抽象
3. 可扩展性好

设计方法：
1. 限制计算模型的复杂度，减少表达难度
2. 优化对网络资源的使用，计算在本地进行
3. 使用冗余计算来提高可靠性与响应速度