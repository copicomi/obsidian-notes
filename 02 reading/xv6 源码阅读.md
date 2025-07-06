## 启动部分

### 阅读文件：
- `kernel/`
	1. `proc.h` 
		- 定义 `context`，`cpu`，`proc`，`trapframe` 结构体
	2. `defs.h`
		- 声明各个文件的核心函数
	3. `entry.S`
		- 初始分配 `CPU` 栈，随后启动 `start` 初始化内核
	4. `start.c`
		- 初始配置，切换到 `kernel mode`，启动时钟中断，进入`main.c`
	5. `main.c`
		- 初始化内核功能，并调度进程，通过`userinit` 进入`user mode` 并创建 `init` 进程
- `user/`
	1. `initCode.S`
		- 引导进入 `init` 程序
	2. `init.c`
		- 系统准备完成后，启动`shell`

---
### `proc.h`

- `context`，`trapframe` 主要包含寄存器内容
- `trampoline.S` 通过调用 `trapframe` 保存进程相关内容，在用户和内核模式之间切换
- `cpu` 维护当前进程、上下文、中断相关、调度相关
- `proc`
	1. 维护状态相关变量
	2. 记录页表、用户栈、进程调度相关信息
---
### `entry.S`

内核数据，载入到 `0x80000000`

为每个 CPU（`hartid`）分配 `4096 byte` 的内核栈，随后调用 `start.c`

---
### `main.c`

如果是第一次开机：
1. 初始化控制台
2. 初始化 虚拟内存系统
3. 初始化 内核进程表、中断向量、trap 向量
4. 初始化 IO设备、文件系统、虚拟磁盘
5. 初始化 用户进程， 通过`userinit()` 进入`user mode` 并创建 `init` 进程
6. 标记为已经成功启动
否则是启动一个 CPU

最后调用 `scheduler`，调度进程

---
### `initcode.S`

内核启动后，执行第一个进程，进入 `user mode` 

执行 `exec(init, argv)`，进入`init`用户进程

---
### `init.c`

启动控制台，分配三个文件描述符

随后调用 `fork()` ：
- 子进程负责 执行 `shell` 
- 父进程负责 保证子进程意外退出时，重启`shell`

成功执行 `shell` 后，系统启动，用户可以与 OS 交互

---

### 总结：运行逻辑

#### 0. 硬件初始化 
> 系统启动时，从 ROM 中读入初始代码，将 kernel 代码载入物理内存
> 现在系统处在 *机器模式（machine mode）* 下，权限最高，未启动地址转换
> 我们将 `entry.S` 放在内核代码的起点，就位于`0x80000000`
> 随后 CPU 从 `$pc` 读取指令，进入`entry.S`

#### 1. entry

> `entry.S` 创建一个新的 C 栈，以执行 C 代码，随后进入 `start.c`

#### 2. start

> 切换到 `kernel mode` ，初始化时钟中断，进入到内核，调用 `main.c`
> **machine part 结束** 

#### 3. main

> 内核的主体部分
> 初始化内核的各个功能（进程、trap、文件系统、虚拟地址、分页），最后用 `userinit()` 启动第一个用户进程 `initcode.S`，进入到`user mode`
> **kernel part 结束** 

#### 4. initcode & init
> 执行 `exec(init)`
> 随后 `init.c` 启动 `shell`，系统成功启动，用户开始交互
> **user part 结束**

至此，xv6系统成功初始化，开始运行

---
## 内存部分

### 阅读文件
1. `kernel/`
	- `memlayout.h`
	- `vm.c`
	- `kalloc.c`
	- `riscv.h`
	- `exec.c`
---
### `memlayout.h`

包含内存相关的宏定义，这里只看和 kernel、user 有关的

1. `KERNBASE` 内核基址 `0x80000000`
2. `PHYSTOP` 物理内存上限 `KERNBASE + 128*1024*1024`
3. `TRAMPOLINE` 存放在虚拟内存的顶部 `MAXVA - PGSIZE`
	-  user 和 kernel 共享本页内容
4. `KSTACK(p)` 内核栈映射 `TRAMPOLINE - ((p)+1)*2*PGSIZE` 
	- `+1` 避开顶部的两页
	- `*2` 考虑哨兵页，给栈的每一个页都设定一个虚拟的`guard page`
	- 实际上 `guard page` 只存在于 虚拟内存中，并不使用 到 物理内存的映射，不占用物理空间
6. `TRAPFRAME` 每个进程独有，位于`TRAMPOLINE`正下方
	- `TRAMPPOLINE - PGSIZE`

---
### `vm.c`

**用于管理内存页表**

起初 `main` 调用 `kvminit()` 初始化 **内核页表**

`kvmmake()`创建直接映射的 kvm 页表
- 调用 `kalloc` 分配物理空间
- 随后调用 `kvmmap` 向页表 加入 系统功能相关的直接映射
	- `kvmmap` 调用 `mappages` 
	- 只在 `booting` 时运行，不刷新 TLB，不启用分页系统
- 然后调用`proc_mapstacks` 对每个进程分配 内核栈

---
#### `mappages(pagetable. va, size, pa, perm)
`
强制 va、pa 按 4096 字节对齐
遍历 `[va, va+size]` 的每一块内存：
1. 用 `walk` 查找 `pagetable`中 va 对应的 PTE 地址
	- `walk` 开启 `alloc` 参数，意味着插入记录时，可能会申请新的多级页表
2. 随后更新 `*pte = PA2PTE(pa) | perm | PTE_V`
	1. `pa` 映射部分
	2. `perm | PTE_V` 状态位

#### `walk(pagetable, va, alloc)`

迭代查找 三个 LEVEL 的页表
如果 `alloc` 打开，查找失败时，就调用`kalloc` 分配页面，更新多级页表
使用 `PX(level, va)` 得到对应的页表偏移量
最后返回 `&pagetable[PX(0, va)]` PTE 指针

---
`main` 随后调用 `kbminithart` 启用分页功能，将根页表的地址存入寄存器

---

### `kalloc.c`

**用于管理和分配空闲页面**

采用头链表结构管理空闲页面，*先进后出式结构*

`main` 调用 `kinit()` 初始化空闲页面，对所有内存页调用`kfree`

`kfree` 清理页面原内容，并加入空闲列表

`kalloc()` 从空闲列表中取出一个空闲页

---
### `riscv.h`

只看和内存有关的部分

定义了一些地址解析和转换的宏定义(如`PA2PTE(pa)`)

---
### `exec.c`
---
### 揭示一块内存页的生命周期

初始化阶段
- 内核采用直接映射，将设备分配到物理地址
- `kinit` 将物理地址存入链表，这样可以避免打开*地址翻译功能*后，难以分辨要取的是物理还是虚拟地址
- 由链表存储，也表示我们不关心空闲页分配的顺序

---

> user mode 下的虚拟地址是如何工作的？
> user mode 和 kernel mode 下的地址表示一致吗？
> kernel mode 下的地址是否也需要被翻译？

按道理说，usermode 的地址转换由 riscv硬件完成（QEMU 模拟），
但是既然一切工作都由硬件完成，那么我们又怎么能通过修改内核实现新功能？

因为进程中的地址是隐式访问的，我们必须要知道这种虚拟地址转换是怎么完成的

#### 系统启动
此时处于 machine mode，分页未启用
首先 free 全部物理内存，存入kalloc链表
随后 建立内核与设备的直接映射，确保内部代码能直接执行

kernel mode 启动后，回答上面的问题
>  riscv 硬件只负责地址转换的部分，并不负责页表维护
>> - 工作过程：
>> 	 1. 获取虚拟地址，从 `satp` 寄存器得到根页表地址
>> 	 2. 查看状态位，如果是 `PTE_V`认为页表有效
>> 	 3. 如果 `PTE_W | PTW_X | PTW_R `为真，认为该页是叶子节点，否则继续迭代查询下一级页表
>> 	 4. 到达叶子节点，加上 va 的 offset，翻译完成
>> 	 5. 从翻译出的 pa 读取数据
>
> 内核负责维护页表映射的部分
>> 功能：
>> - 初始化(machine mode)
>> 	1. kalloc 释放全部物理内存后，加入空闲链表
>> 	2. 分配内核根页表时，直接调用 `kalloc()`，使用得到的物理地址即可（已得到直接映射）
>> 		- 这些直接映射的部分虽然占用了一块虚拟空间，但是 PTE_U 标志未设置，用户访问会被禁止
>> - 页表维护（kernel mode）
>> 	1. `mappage` 接受 va 和 size，用 `walk` 查找和创建三级页表，最后将映射插入页表

#### 一块内存的事件

1. 申请：
	- 两个起点：`sbrk() syscall` 与 `proc create`
	- 最后都会到达 `uvmalloc()`
		- 遍历每一块页，尝试 `kalloc` 和 `mappage`
		- 创建多级页表时，使用`walk`申请空间
	- 插入页表后，在 user mode 下可以正常使用了
2. 进程释放：
	- 首先调用`uvmummap`, `kfree` 掉申请的用户空间
	- 随后调用`freewalk`，递归`kfree`掉所有的页表空间

---
	
---
## Trap部分
#### 阅读文件：

1. `trampoline.S` 
2. `trap.c`
3. `riscv.h` 一些汇编命令的宏

---

三种 Trap 情况 ：
1. syscall
2. 异常
3. 设备中断（I/O）
---
### `trampoline.S`

`uservec` 和 `userret` 的汇编代码

`trampoline` 页被内核和用户共享，该代码是公共的

---

### `trap.c`

`usertrap` 和 `usertrapret` 的 c 代码
## 进程部分

### 阅读文件
- `proc.c` 进程相关代码
- `swtch.S` 上下文切换代码
	- 仅仅是寄存器的保存与加载，不单独讲了
	- 涉及寄存器操作，故采用汇编

### `proc.c`
从内核 `main`初始化结束后，调用 `userinit` 创建进程

#### `userinit()`
调用 `allocproc()` 创建新进程，随后指名进入 `initcode`

#### `allocproc() / freeproc()`

先查找进程表 `proc table`，找到 `unused proc` 才创建进程

- `kalloc` 一个 `trapframe`
- 申请一个空的 `user page table`
	- `proc_pagetable()` 
		- 调用 `uvmcreate()` 初始化根页表
		- 随后为 `trapframe` 和 `trampoline` 插入 va 映射
			- 在此过程中 `mappage()` 创建二级和三级页表
- 设置 `context` （~~?~~调度相关） 
	- 设置`%era = forkret` ，`%sp = kstack`
	- 进程调度时，`swtch`会返回到新进程上次中断的地方
	- 但是对于一个新进程，它是没有这种概念的，没有开始怎么谈中断呢
	- 因此这里引入 `forkret`作为 **伪造的新进程**，这样执行`swtch`后就可以跳转到用户代码开头
	- **`forkret`只在这里被使用**

该函数负责申请一个空进程，为上层调用者所使用

`freeproc()` 就是简单释放进程占用的资源

---
#### `sleep/wakeup 逻辑`
通过对进程状态的检查与修改来实现

要查找可运行的进程，就遍历进程表，找到`RUNNABLE`

要唤醒对应的进程，就遍历进程表，找到 `SLEEPING`

通过遍历状态表，实现进程间的关联

要休眠，就设状态为`SLEEPING`
要停止，就设状态为`ZOMBIE`
要唤醒，就设状态为`RUNNABLE`，准备被调度器选走

通过修改状态，来避免被其他无关的进程所检查

- `sleep()`  修改状态，转让CPU给`scheduler`
- `wakeup()` 查进程表，唤醒对应休眠进程
---
#### `wait/exit 逻辑`
该套逻辑确定父子进程的串行执行，同时管理进程资源的释放
- `kill()` 
	- 设置`kill`标志
	- 内核代码运行时，自行检查`kill`标志，决定是否`exit()`
- `exit()` 
	- 关闭打开的文件资源
	- 转移子进程到`init`
	- 将状态设为`ZOMBIE`
	- 转让CPU给 `scheduler`
- `wait()`
	- 死循环
	- 查进程表
	- 找到状态为`ZOMBIE`的子进程
	- 清除其进程资源
	- 将状态设置为`UNUSED` 

子进程的资源释放必须依赖父进程的`wait()`

而xv6 的进程是由`init`与`shell` 执行`fork()`得到的，因此所有进程都可以交给`init`进程释放资源

故xv6将子进程的父指针重定向到`init`进程，`init`的任务就是无限循环，执行`wait()`清理结束进程

同时采用`kill`功能，在系统出错时及时退出

---
#### `scheduler逻辑`

进程调度的核心机制，在于 **上下文切换**

- `swtch()` 通过汇编保存和加载寄存器，以切换上下文
- `sched()` 负责切换到调度器进程
- `scheduler()` 负责查找`RUNNABLE`进程，切换到新进程

上面三个函数就是基本的切换逻辑

>为什么可行呢？

通过切换寄存器，我们改变了 `%ra` ，实现代码上下文的切换，`ret`后将会来到新地址

下面阐述其设计思路：
- 核心是 `swtch` 带来的 **保存进程状态** 功能
-  `swtch` 前后的代码运行，虽然是连贯的，却从属于不同的进程
	1. 寄存器内容不同
	2. `ret` 返回地址不同
- `sched` 与 `scheduler` 扮演临界区的作用，进程在这里中转，为切换进程提供接口
- 这样的机制，为进程提供了自己独占 CPU 的假象——调度器再返回时，寄存器没变化、指令位置没变化、返回的函数地址没变化（如果自己独享一块页表，甚至内存也没变化）
- 进程会照常执行，而调度过程对用户则是透明的

`scheduler` 的`context` 保存在 CPU 内部，每个 CPU 都有单独的调度器进程

---
特别注意的是，`swtch`是从一个进程切换到另一个进程，这个新进程显然是之前从 `swtch` 中断，被迫切换到其他进程（也就是已经执行了一半）

如果这是一个新进程，从没有执行过，那么就不存在对应的`context`了

因此我们在`allocproc()`中会将`p->context`初始化为指向 `forkret` 的内容，伪造了一个 **正在执行`forkret`的进程** ，`forkret`随后调用一个伪造的 `usertrap`，返回到用户进程的首条指令，这样就切换到了一个全新的进程

### Think

所有进程中，第一个进程由`userinit` 分配 `pid`，其余都是从`fork`得来的

---
## 锁部分

`spinlock.c`

一个`struct lock` 记录以下信息：
1. 锁状态
2. lock name
3. 被哪个 CPU 所持有

处理锁的要点：
1. 关闭中断
	- 这么做是防止`acquire`被计时器中断，由另一个进程再次获取同一把锁，导致死锁
	- 重复获取同一个锁时，引发`panic`
2. 原子指令，进行锁操作
3. 告诉编译器不要重排指令

---
## 文件部分
### disk

该层分为
0. super 区
1. inode 区
2. 位图区
3. 数据区

通过查找位图区，实现 `block` 的申请与释放

---
### buffer cache
缓存常用块的写入，一次性与磁盘集中交互
`bio.c`

采用双向链表结构管理缓存块结构

当上层调用 磁盘 IO 时，首先会向缓存区发起读写请求，获取一块`block`空间，如果不在缓存中就通过`virtio_disk_rw()`与磁盘进行交互，更新缓存区 block

该链表按照最近使用时间排序节点

---
### 日志

系统崩溃时需要保持文件操作的原子性

xv6先将磁盘操作记录到 log 中，再执行读写操作，崩溃时根据日志恢复数据

日志同样存在磁盘中，不过会先在内存区的缓冲区中积累，随后整块地写入磁盘，保证日志写入的原子性

读写操作两端使用`begin_op()`和`end_op()`函数

在此之间的任何写操作会先写入到`log_write()` 中

`end_op()` 负责将 `log` 中的写操作 `commit`，随后清空 `log`

`initlog()`负责在系统崩溃重启后，检查`log`内容，重放操作，恢复磁盘

---

### Inode 缓存

由于不可能把所有的 `inode` 都放入内存，因此维护一个缓存区，只引入当前程序中，存在
指针引用的 `inode`

磁盘中的`dinode`才是原数据

使用 itable数组作为缓存区
`ialloc` 查找空闲的 inode 空间
`iget` 和 `iput` 负责与 disk 的交互
`ilock` 和 `iunlock` 维护锁

---

### inode

`dinode`存储在磁盘上，维护文件的元信息
1. 类型
2. 大小
3. block地址（直接或间接寻址）

`Bmap()`负责解析 `block` 地址
`itrunc()` 负责释放`block`，修改对应的`dinode`

`readi()`和`writei()`de通过`Bmap()`获取块地址

---
