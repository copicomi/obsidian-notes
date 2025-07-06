#### 阅读NEMU源码框架
```
nemu
···
├── src                        # 源文件
│   ├── cpu
│   │   └── cpu-exec.c         # 指令执行的主循环
│   ├── device                 # 设备相关
│   ├── engine
│   │   └── interpreter        # 解释器的实现
│   ├── filelist.mk
│   ├── isa                    # ISA相关的实现
│   │   ├── mips32
│   │   ├── riscv32
│   │   ├── riscv64
│   │   └── x86
│   ├── memory                 # 内存访问的实现
│   ├── monitor
│   │   ├── monitor.c
│   │   └── sdb                # 简易调试器
│   │       ├── expr.c         # 表达式求值的实现
│   │       ├── sdb.c          # 简易调试器的命令处理
│   │       └── watchpoint.c   # 监视点的实现
│   ├── nemu-main.c            # 你知道的...
│   └── utils                  # 一些公共的功能
│       ├── log.c              # 日志文件相关
│       ├── rand.c
│       ├── state.c
│       └── timer.c
···
```
核心代码是`nemu/src`部分
- 从主函数`nemu-main.c`入手理解框架代码
	- 发现调用了`engine_start()`和初始化函数
	- 使用 `grep` 命令找到引擎启动函数位于`/engine/interpreter/init.c`
		- 进一步查看`init.c`，又调用了`cpu_exec(-1)`和`sdb_mainloop()`
		- `cpu_exec()`用于执行汇编指令，`sdb`则负责实现调试功能
- 阅读`/monitor/sdb/sdb.c`源码
	该程序分为三部分：
	1. 循环主函数`sdb_mainloop()`
		该函数实现了`cmd`命令的字符串读入与`token`分解，后面再作说明
	2. 存储调试指令的结构体数组`cmd_table`
		```c
		static struct {
			const char *name;
			const char *description;
			int (*handler) (char *);
		} cmd_table [] = {
			{ "help", "Display information about all supported commands", cmd_help },
			{ "c", "Continue the execution of the program", cmd_c },
			{ "q", "Exit NEMU", cmd_q },
		
			/* TODO: Add more commands */
		
		};
		```
		- name : `char*`
			- 用于存储指令名称
		- description : `char*`
			- 用于help指令的实现
		- handler : `int* (char*)`
			- 一个函数指针，接受一个`char*`参数，返回`int`
			- 用于重定向，处理调试指令
			- ``if (cmd_table[i].handler(args) < 0) { return; }``
	3. 负责实现指令和输入输出的其他函数  
- 阅读`cpu_exec(n)`函数，该函数模拟cpu工作
	- `n`为指令条数
	- 指令执行前后，用`switch`检查cpu运行状态，判断是否应该终止运行，是否正常退出
	- 调用`execute()`执行指令，运行逻辑如下：
		- 逐条取出指令，指令类型为`Decode`
			- 执行指令`exec_once()`
			- 检查正常运行，否则退出
			- 进行I/O
			- 写入内存`IFDEF(CONFIG_DEVICE, device_update())`
	- 阅读 `exec_once()`
		- 从寄存器`cpu.pc`读出指令地址
		- 用ISA执行指令，调用`isa_exce_once()`
		- 更新寄存器`pc`
- 阅读`isa_exec_once()`，位于`isa/riscv32/inst.c`
	- 调用`inst_fetch()`获取指令
	- 返回`decode_exec()`的运行结果
关于ISA的实现部分先跳过 **// TODO //**
##### 修bug
用`make run`/`make gdb`运行调试NEMU
- 发现直接输入q，NEMU返回错误值`1`
用[[gdb]]调试可以发现：
- NEMU的运行状态用`nemu_state`全局变量储存，结束时检查该状态是否合法（`NEMU_END`或`NEMU_QUIT`)
- 输入`c` `q`时先运行客户程序，再退出NEMU是正常的
- 只输入`q`却错误退出
- 调试发现`c`运行到客户程序末尾时，将状态标记为`NEMU_END`可以正常退出
考虑用[[gdb]]的`watch`命令监视`nemu_state`,
- 发现`q`命令从未修改`nemu_state`
修改`sdb.c`中的`cmd_q()`函数，添加`set_nemu_status(NEMU_QUIT)`即可
