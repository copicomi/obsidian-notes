#input #tool #makefile #fleeting 
https://seisman.github.io/how-to-write-makefile/
## 概述
- Makefile关系到整个工程的编译规则
- 优点：自动化编译
	- 只需要一个make命令，整个工程自动编译
- 程序的编译与链接
	- **编译**：
		- 把源代码编译成中间代码文件，UNIX下是`.o`文件，Windows下是`.obj`文件，这个动作叫做编译（compile）
		- 关心头文件的所在位置
		- 检测语法和函数变量
	- **链接**：
		- 把大量Object File合成可执行文件
		- 只需要中间文件，不关心源代码
		- 把中间目标文件打包，就是`.lib`文件（Windows）或者`.a`文件（UNIX），即库文件 
## makefile介绍
#### makefile的规则
```
target ... : prerequisites ...
	recipe
	...
	...
```
- **target** :
	- 目标文件 `.o`
	- 可执行文件 
	- 标签（label）
- **prerequisites**：
	- 生成target所依赖的文件
	- target
- **recipe**:
	- 该target执行的shell命令
#### 一个示例
	
>在这个示例中，我们的工程有8个c文件，和3个头文件，我们要写一个makefile来告诉make命令如何编译和链接这几个文件。我们的规则是：
>1. 如果这个工程没有编译过，那么我们的所有c文件都要编译并被链接。
>2. 如果这个工程的某几个c文件被修改，那么我们只编译被修改的c文件，并链接目标程序。
>3. 如果这个工程的头文件被改变了，那么我们需要编译引用了这几个头文件的c文件，并链接目标程序。
```
edit : main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o
    cc -o edit main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o

main.o : main.c defs.h
    cc -c main.c
kbd.o : kbd.c defs.h command.h
    cc -c kbd.c
command.o : command.c defs.h command.h
    cc -c command.c
display.o : display.c defs.h buffer.h
    cc -c display.c
insert.o : insert.c defs.h buffer.h
    cc -c insert.c
search.o : search.c defs.h buffer.h
    cc -c search.c
files.o : files.c defs.h buffer.h command.h
    cc -c files.c
utils.o : utils.c defs.h
    cc -c utils.c
clean :
    rm edit main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o
```
其中`recipe`一定要以一个缩进为开头

#### make是如何工作的
`make`命令逐层检查依赖关系，若依赖可靠（修改时间早于target），则执行shell命令，否则递归地检查依赖，直到查无可查，此时make停止工作
#### makefile中使用变量
在makefile文件开始处定义即可（类似于宏）
```
objects = main.o kbd.o command.o display.o \
    insert.o search.o files.o utils.o

edit : $(objects)
    cc -o edit $(objects)
clean :
    rm edit $(objects)
```
要想增加`.o`文件，只需修改 `object`

#### 让make自动推导
make看到`.o`文件作为target时，会自动把对应的`.c`文件加入依赖，这就是make的“隐式规则”
```
main.o : defs.h
kbd.o : defs.h command.h
command.o : defs.h command.h
display.o : defs.h buffer.h
insert.o : defs.h buffer.h
search.o : defs.h buffer.h
files.o : defs.h buffer.h command.h
utils.o : defs.h
```
#### makefile的另一种风格
```
objects = main.o kbd.o command.o display.o \
    insert.o search.o files.o utils.o
    
$(objects) : defs.h
kbd.o command.o files.o : command.h
display.o insert.o search.o files.o : buffer.h
```
多个`.o`对应一个`.h`，但不推荐
#### 清空目录的规则
```
.PHONY : clean
clean :
    -rm edit $(objects)
```
不要放在文件的开头，不然会编程make的默认目标
`.PHONY`表示`clean`是一个“伪目标”
#### Makefile里有什么？
1. 显式规则
2. 隐式规则
3. 变量定义
4. 指令
5. 注释
#### Makefile的文件名
最好使用`Makefile`
#### 包含其他Makefile
可以使用`include`指令包含其他的Makefile，语法为``include <filenames>...``
#### 环境变量MAKEFILES
不建议使用，会影响所有的Makefile
#### make的工作方式
1. 读入所有的Makefile。
    
2. 读入被include的其它Makefile。
    
3. 初始化文件中的变量。
    
4. 推导隐式规则，并分析所有规则。
    
5. 为所有的目标文件创建依赖关系链。
    
6. 根据依赖关系，决定哪些目标要重新生成。
    
7. 执行生成命令。