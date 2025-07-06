## super page

`uvmalloc` 调用 `kalloc` 分配普通页

`kalloc` 每次申请 512 字节

在 `uvmalloc` 的循环中，将待分配空间与 `2M` 比较，决定是否分配超级页面（注意地址对齐）

`superalloc` `superfree`

两部分逻辑
1. `alloc` 与 `free` 逻辑
	1. 预留 16 个 2M 超级页，单独处理 super 链表 
	2. 其余逻辑模仿 `kalloc`
2. 何时使用 超级页
	1. 在 `uvmalloc` 的循环中判断以下条件
		1. 待申请足够大
		2. 地址对齐

super page 的 offset 为 21 位，只需查询 二级页表

superpage 的
1. 申请
2. 访问
3. 释放
4. 映射插入
5. fork 处理

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

要实现超级页，我们只需要保证页表映射和释放不出问题
即 `mappage` ,`walk` 的行为支持超级页

要实现这些功能，只需要申请普通页前，先判断超级页的条件是否满足即可
而查询功能，在第二级页表设计标志位即可实现

但是free超级页时，`uvmunmap` 采用 `walk` 查找物理页，返回时只有该页的起始地址与标志位，这样导致我们不能分辨到底是普通页还是超级页，需要扩展一个标志位`PTE_S`

`mappage` -- `walk` 是一整套逻辑



---
设计文档：

我们尽量让两种页面共用同一套逻辑
遍历多块地址空间的逻辑，都通过`PTE_S`判断 size
#### `vm.c`

- `uvmalloc()`
	- 添加判断是否加入超级页的逻辑
	- 调用`superalloc` 与 `kalloc`
	- 调用`mappage`
		- 在参数 `perm` 中通过扩展位体现超级页
		- 将分类讨论封装到`mappage`内部
	- 调用`uvmdealloc`
- `mappage()`
	- 判断参数`perm` 是否带有`PTE_S`
- `walk()`
	- 将第三个参数拓展为 32比特的 `option`
	- 第 0 位为 `alloc`，第 1 位为 `superpage`
- `uvmunmap()`
	- 将参数 `dofree` 扩展为 `option`
	- 第 0 位为 `dofree` ,第 1 位为 `superpage` 

---

不再将不同逻辑杂糅到同一个循环内部，特殊讨论 walk 中 alloc + superpage 的情况

通过`fork()` 前的测试 

---
`fork()` 调用 `uvmcopy()`  实现进程空间复制