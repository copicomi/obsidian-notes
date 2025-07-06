[[OSTEP 操作系统导论]]
# 并发

## 概念

OS 中存在多线程并行，对于共享资源会出现副作用

锁、原子指令、条件变量

## 机制：锁

#### 评价标准：
1. 互斥
2. 公平
3. 性能
### 锁实现
#### 中断锁
1. 特权模式，权限过大
2. 不支持多处理器
3. 中断丢失
4. 效率低

#### 简单标志锁
```c
while (flag == 1) ;
```
```c
A --- while (flag == 1)
#
B --- while (flag == 1)
B --- flag = 1
#
A --- flag = 1
```

TEST 之后未 SET 就被中断，导致异常
不互斥


#### 测试并设置指令

不让中断发生在错误位置，用硬件实现原子指令
```c 
int testAndSet() {
	*old = new_val; 
	return old_val;
}
```
```c
void lock() {
	while (testAndSet(&flag, 1) == 1) ;
}
```
把检查锁和赋值锁组合成原子指令，有效互斥

#### 自旋锁
正确、不公平、性能一般

#### 比较并交换指令

``` c
int compareAndSwap() {
	if (*p == expect_val) 
		*p = new_val;
	return old_val;
}
void lock() {
	while (compareAndSwap(&flag, expect_val=0, new_val=1));
}
```
#### 链接加载、条件式存储指令

``` c
int loadLinked() {return *p;}
int storeConditional() {
	if (no one updated *p since loadLinked()) {
		*p = new_val;
		return 1;
	}
	else 
		return 0;
}
void lock() {
	while (loadLinked(flag) ||
		!storeConditional(flag, 1));
}
```
#### 获取并增加指令

``` c
int fetchAndAdd() {
	*p += 1;
	return old_val;
}
```
给进程分配编号，解锁用编号自增实现

### 性能优化

锁自旋时，仅仅是该 pc 计数器不增加罢了，但物理上仍占用 CPU 资源

因此，自旋时，进程调用 `yield()` 主动放弃 CPU

但一个进程可能不断主动放弃 CPU，导致饥饿

采用休眠队列，实现这种公平调度


## 基于锁的并发数据结构

1. 并发计数器
2. 并发链表
3. 并发队列
4. 并发散列表

## 条件变量

检查条件满足后，才继续运行

### 生产者/消费者问题

