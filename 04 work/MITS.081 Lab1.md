## primes

管道实现埃氏筛

进程伪代码
```
从 pipe1 read 整数 p
print p
loop :
	从 pipe1 read 整数 n
	如果 n 不能整除 p
		向 pipe2 write 整数 n
```

将上述进程称为 `pipe_thread`

---

`primes.c`伪代码
```
创建 pipe1 ,写入 2~230
创建 pipe2

:LOOP

pipe_thread(pipe1, pipe2)

如果管道 pipe2 仍有数据:
	pipe1 = pipe2
	pipe2 = 新pipe
	GOTO LOOP
```

---

采用 `while(1)` 循环，当 `pipe_thread` 读入 `p` 失败时，才 `break`

`primes.c`伪代码
```
创建 pipe1 ,写入 2~230
创建 pipe2

while(1) :
	从 pipe1 read 整数 p
	如果读取失败，跳出循环
	print p
	
	loop :
		从 pipe1 read 整数 n
		如果读取失败，跳出循环
		如果 n 不能整除 p
			向 pipe2 write 整数 n
	
		pipe1 = pipe2
		pipe2 = 新pipe
```

---
由于进程是并行的，所以不能迭代更新管道，管道不能复用，只能在结束的时候关闭管道流

`primes.c`伪代码
```
创建 pipe1 ,写入 2~230

------------------------

while(1) :
	从 pipe1 read 整数 p
	如果读取失败，关闭pipe1，跳出循环
	print p

	创建 pipe2
	
	loop :
		从 pipe1 read 整数 n
		如果读取失败，关闭pipe1，跳出循环
		如果 n 不能整除 p
			向 pipe2 write 整数 n
	
	
```
---
现在考虑`fork()`的调用时机
一对`pipe`对应一个`fork`
`fork()` 后，父进程用新的`pipe`与子进程通信
两个进程应当分别独立进行

- 父进程成功读到 `p` 后，`fork`一个子进程，并新建管道与之通信
- 随后父进程正常工作，处理上层的数据流
- 但是子进程应该也有读取 `p` 的部分，`fork()`之后必须`GOTO`到前面的代码

`primes.c`伪代码
```
创建 pipe1 ,写入 2~230

------------------------
Begin_Pipe_Thread:
	从 pipe1 read 整数 p
	如果读取失败，关闭pipe1，跳出循环
	print p

	创建 pipe2
	fork()
	
	if (getpid() == 0): //子进程
		GOTO Begin_Pipe_Thread
	
	loop :
		从 pipe1 read 整数 n
		如果读取失败，关闭pipe1，跳出循环
		如果 n 不能整除 p
			向 pipe2 write 整数 n
	
	
```

---
用一个`bool`变量优化掉`GOTO`

`primes.c`伪代码
```
创建 pipe1 ,写入 2~230

------------------------
while (child_flag == 1)

	child_flag = 0 // 保证子进程只循环一次

	从 pipe1 read 整数 p
	如果读取失败，关闭pipe1，跳出循环
	print p

	创建 pipe2
	fork()
	
	if (getpid() == 0): //子进程
		child_flag = 1  // 抹掉从父进程 copy 来的标记
	
	loop :
		从 pipe1 read 整数 n
		如果读取失败，关闭pipe1，跳出循环
		如果 n 不能整除 p
			向 pipe2 write 整数 n
	
	
```
---
现在修改，进程间 的 `pipe` 继承关系
`fork` 之后，把父进程的 `pipe2` 给到 子进程的 `pipe1`

`primes.c`伪代码
```
创建 pipe1 ,写入 2~230

------------------------
while (child_flag == 1)

	child_flag = 0 // 保证子进程只循环一次

	从 pipe1 read 整数 p
	如果读取失败，这就是流的结尾了，关闭 pipe1，直接exit()
	【防止反复释放 pipe】
	print p

	创建 pipe2
	fork()

	【这一段是为了重置 child 的上下文信息】
	if (getpid() == 0): // 子进程
		child_flag = 1  // 抹掉从父进程 copy 来的标记
		pipe1 = pipe2   // 设置管道

	
loop :
	从 pipe1 read 整数 n
	如果读取失败，关闭pipe1，跳出循环
	如果 n 不能整除 p
		向 pipe2 write 整数 n

关闭 pipe2
exit()
```

---
一个右管道 `pipe2` 对应一个进程，除了最后一个进程，正常进程最后都会关闭写端口【写端口安全释放】

所有进程读不到数字后，都会关闭读端口，包括最后一个进程【读端口安全释放】

注意起初写入所有数字的端口，该管道需要在最后关闭

`primes.c`伪代码
```
创建 **zpipe0** ,写入 2~230
关闭 pipe0 的写端口

令 pipe1 = pipe0
------------------------
while (child_flag == 1)

	child_flag = 0 // 保证子进程只循环一次

	从 pipe1 read 整数 p
	如果读取失败，这就是流的结尾了，关闭 pipe1 读端口，直接exit()
	【防止反复释放 pipe】
	print p

	创建 pipe2
	fork()

	【这一段是为了重置 child 的上下文信息】
	if (getpid() == 0): // 子进程
		child_flag = 1  // 抹掉从父进程 copy 来的标记
		pipe1 = pipe2   // 设置管道

	
loop :
	从 pipe1 read 整数 n
	如果读取失败，关闭pipe1读端口，跳出循环
	如果 n 不能整除 p
		向 pipe2 write 整数 n

关闭 pipe2 写端口
exit()
```
---
上面的代码，存在pipe缓存阻塞问题，不能一次性写入全部数字，最好先fork，再进行写入工作

---
完成初始化修改后，运行，发现pipe2的分配失败
应该是没有及时释放资源导致的

不能把读入和写入放在两个循环体内，二者必须并行

---
重构代码，不再采用迭代，采用递归式

```
main :
	输入数字到管道0

primes :
	读取p
	if 流结束：
		关闭 rd，return
	else：
		父进程创建新管道，然后正常运行
		fork 子进程调用 primes 然后 return
	
	循环读取 n
```