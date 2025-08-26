# 阅读背景
6.5840 2025 Lecture 3: Primary/Backup Replication  
6.5840 2025 讲座 3：主/备份复制

Today
  Primary/Backup Replication for Fault Tolerance
  Case study of VMware FT (2010), an extreme version of the idea  
今天主要讨论主/备份复制，以 VMware FT（2010）为例，这是该想法的一个极端版本  
  
Why read this paper?
  A clean primary/backup design that brings out many issues that come
  up over and and over this semester
    state-machine replication
    output rule
    fail-over/primary election
  Impressive you can do this at the level of machine instructions
    you can take any application and replicate it VM FT
    all designs later in semester involve work by the application designer  
为什么要读这篇论文？一个干净的主/备份设计，在本学期中反复出现的问题：状态机复制输出规则、故障切换/主节点选举 令人印象深刻，你可以在机器指令级别实现这一点，你可以复制任何应用程序，VM FT，本学期后期的所有设计都涉及应用程序设计者的工作  
  
Goal: high availability
  even if a machine fails, deliver service
    i.e., no down time even if a machine fails
  approach: replication  
目标：高可用性，即使机器故障，也能提供服务，即即使机器故障，也不会停机，方法：复制

用于主从备份系统状态，保持系统可靠性

# 1 Introduction
常规的主从备份，需要通过网络发送内存的所有内容，针对数据本身进行备份，占用大量带宽
而本论文提出的设计，只发送输入，在物理机上进行操作重放
  相当于发送log，在备份机上进行replay，达到复制的效果
  在此基础上，用状态机描述备份状态，用虚拟机平台抵消硬件影响

# 2 Basic FT Design
本节叙述保证数据一致性的核心design
主从机运行在VM上，input/output由primary机管理，将input通过channel告知backup机，进行replay

三个要点：
1. 如何保证backup数据的正确性
2. 如何处理信息 loss
3. 如何处理execution fail

## deterministic replay
如何保证 backup 是一个DFA？
  由primary发送cpu执行的每一条指令
  影响确定性的因素，如 IO 中断、时钟等信息，也一同进入log channel

## FT 协议
通过channel进行发送，使得 backup机 可以即时响应状态变化
如果primary机在进行output后，没有将状态正确备份到backup，就会发生不一致问题
  因此primary会延迟output时机，等待output操作在backup机上正常replay后，确认完成备份，再进行真实的output
  这样，当 primary机 fail 后，backup机可以回溯到两次output间的一个正确备份，从而接管工作
  而这一切对 user 是透明的

TCP解决丢包

## failure探测
通过 heartbeat 等方式探测另一方是否断线，从而决定是否 go live
当 backup VM 接管 primary VM 时，先消耗完全部的 replay log，再通告 primary VM 的新 MAC 地址

split brain：
  如果heartbeat消失，仅仅是由于网络因素，造成了假死，那么两个 VM 都会认为自己是 primary VM
  此时需要在共享disk上，执行一条原子性指令，用于测试另一 VM 是否存活，以避免 split brain
