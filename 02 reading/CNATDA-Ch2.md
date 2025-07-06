#计算机网络 
#reading
 [[计算机网络自顶向下]]

# Chapter 2 应用层
##### INTRO
- 应用层概念
- 应用程序，TCP/UDP
- 套接字开发
## 2.1 网络应用原理
### 2.1.1 体系结构
- 客户-服务器体系结构
	- 客户之间不直接通信
	- 服务器是公开的
- 对等（P2P）
	- 效率高，但较为分散
### 2.1.2 进程通信
套接字（[[socket]]）是一个软件接口，进程通过socket向网络进行通信

进程发送的信息需要标记出
1. 主机的地址
2. 接收进程在主机中的标识符

### 2.1.3 可用的运输服务
服务需求指标的分类
1. 可靠数据传输
2. 吞吐量
3. 定时
4. 安全性

### 2.1.4 因特网提供的运输服务

1. TCP服务
2. UDP
3. 不提供能保证定时和吞吐量的服务

### 2.1.5 应用层协议
协议所定义的内容
- 报文类型
- 字段与语法描述
- 字段语义
- 进程何时与如何响应和发送报文
应用层协议是网络应用的一部分

### 2.1.6 将要谈论的网络应用
1. Web
2. email
3. 目录服务
4. 流式视频
5. P2P

## 2.2 [[Web]] 和 [[HTTP]]
### 2.2.1 HTTP 概述

Web术语
- Web页面
- HTML
- URL 由 主机名 和 目标路径 构成
- Web浏览器实现了HTTP的客户端
- Web服务器实现了HTTP的服务端

HTTP定义了请求网页和传送页面的方式
HTTP 采用 TCP
HTTP 不保存来自客户端的状态，是一个[[无状态协议]]

### 2.2.2  非持续连接与持续连接

二者的区别在于 是否始终使用 同一个TCP协议 传输所有的数据

#### 非持续连接的HTTP
工作过程
1. HTTP 客户端 向 目标服务器 发起 TCP连接
2. 经 socket 发送请求报文，包含请求文件的路径
3. 服务端 经 socket 接收请求，封装请求对象，发送响应报文
4. 服务端通知 断开 TCP 连接
5. 客户端接收响应报文，TCP 连接关闭
6. 收到的 HTML 文本，可能包含其他存储在服务端的对象URL，对每个 URL 递归调用上述过程

定义往返时间 [[RTT]] (Round-Trip Time)
此时响应时间为 2 个 RTT 加上 服务端传输 HTML 文件花费的时间

#### 缺点
RTT浪费过多，大量TCP的连接和断开，占用大量资源

#### 持续连接的 HTTP

与上述过程类似，只是不再主动断开 TCP 连接

### 2.2.3 HTTP 报文格式

#### 请求报文
- 由 ASCII 文本书写
- 每行结尾为 `\r\n`
- 结构
	1. 请求行
		- 方法 --- URL --- 版本
	2. 首部行（若干）
		- 首部字段名 --- 值
	3. 实体行
		- 实际需要传输的文件内容

#### 响应报文
- 结构
	1. 状态行：版本---状态码---短语
		常见状态码
		- 200 OK
		- 301 Moved  Permanently
		- 400 Bad Request
		- 404 Not Found
		- 505 HTTP Version Not Supported
	2. 首部行
	3. 实体行

### 2.2.4 用户与服务器的交互 cookie
由于 HTTP 协议是无状态的，而实际情境中需要针对特定用户提供服务，产生识别与跟踪用户的需求，为此 HTTP 使用了 [[cookie]]

cookie信息通过四个节点工作：
1. 响应报文中的cookie首部行
2. 请求报文中的
3. 用户端自行保存的cookie文件
4. 服务端的后端数据库的cookie记录

cookie并不实际跟踪用户的通讯，它只是在服务端记录了用户的各种操作，存入数据库，等待下次同一cookie登录时，从后端查看历史

这种跟踪是非基于连接的，相当于离线跟踪，即便用户不与服务端连接，服务端也保有用户端的所有状态（因此具有侵犯隐私的风险）

### 2.2.5 Web 缓存
起到常见的缓存作用

工作过程：
1. 客户端 先向 Web缓存器 建立连接，查询有无所需数据
2. 若有，缓存器直接响应即可，否则 缓存器 向 总服务端 建立连接，请求对象，并将其存入缓存，随后再响应客户端的请求

但是缓存器内的对象，很可能已经过期
此时，用 HTTP 向总服务端发起 GET 请求，不要求网页，只传输几条短短的报文，检查最近修改时间即可

### 2.2.6 HTTP/2
- 公布于2015年

主要目标是减小感知时延

HTTP/1.1 通过单一 TCP 连接进行传输，导致[[HOL阻塞]]问题，当时的解决方案是打开多个并行的 TCP 连接，但会占用大量带宽

HTTP/2 的基本目标之一就是摆脱这种并行连接

#### HTTP/2 成帧
新的解决方案是：
把报文切分成小帧，分包传输，在目的地装配起来

这样就避免了大文件阻塞的问题，采用时间上（从帧角度来说）轮转传输的策略，使得每个文件占用的带宽是平等的

#### 响应报文的优先次序和服务器推
允许安排请求的相对优先级

允许服务器为一个请求发送多个响应，提前响应用户可能需要的其余对象，省去了等待用户再次发来请求的时延

#### HTTP/3
基于[[QUIC]]
仍处于草案阶段

## 2.3 因特网中的[[电子邮件]]

现实产品：Segaller 1998

电子邮件是一种**异步通信**媒介

主要组成：
1. 用户代理
2. 邮件服务器
3. 简单邮件传输协议 [[SMTP]]


### 2.3.1 [[SMTP]]

电子邮件的**核心**

工作过程：这里称发送方为Alice，接收方为Bob
1. Alice 调用邮件代理程序，提供目标地址 Bob 与邮件正文，然后指示用户代理 A 发送报文
2. 用户代理 A 把报文发送到邮件服务器 A，进入邮件队列
3. 服务器 A 上的 SMTP 客户发现该邮件，向邮件服务器 B 上的 SMTP 服务器建立 TCP 连接
4. 握手并确立连接后，双方的 SMTP 开始通过 TCP 连接 传输邮件
5. 服务器 B 接收邮件后，将邮件放入 Bob 的邮箱
6. Bob 调用 用户代理，阅读 Alice 发送的邮件

SMTP 即作客户端，也作服务端

观察到下述现象：
- SMTP 不使用中转的邮件服务器，而是在服务器 A 和 B 之间建立直接的 TCP 连接
- 安全性

### 2.3.2 邮件报文格式
首部行 --- 空行 --- 报文体

### 2.3.3 邮件访问协议

Alice 通过将邮件托管到 邮件服务器，避免了接收方不可达的问题，邮件服务器会自动反复提交邮件

Bob 则通过 HTTP 或 IMAP 协议从 他自己的邮件服务器 接收邮件

## 2.4 [[DNS]]：因特网的目录服务

识别主机的两种方式：hostname 与 IP 地址

### 2.4.1 DNS 提供的服务
域名系统 DNS(Domain Name System)
提供主机名向 IP 地址转换的服务

DNS是：
1. 一个由分层的 DNS 服务器实现的 **分布式数据库**
2. 一个使得主机能够查询分布式数据库的 **应用层协议**

DNS服务器：BIND软件的 UNIX 机器
DNS协议 ： UDP、Port53

当客户发出一个请求 URL 时，DNS 工作过程如下：
1. 前提：该主机上运行着 DNS 应用的客户端
2. 浏览器从 URL 中抽取出 hostname，传给 DNS 客户端
3. DNS 客户向 DNS 服务端 发出包含 hostname 的请求
4. DNS 服务端响应 hostname 对应的 IP 地址
5. 浏览器接收到 IP 地址，发起 TCP 连接

其他服务：
1. 解析主机别名
2. 解析邮件服务器别名
3. 负载分配
	- 有时一个 hostname 可能对应多台 服务器 IP 地址，DNS 在这些服务器间循环提供地址（而不是总是响应第一个服务器），在服务器间平衡了负载

### 2.4.2 DNS 工作机理概述
 最简单的一种设计是：全网络部署唯一的 DNS 服务器总部
 高度集中的特点使得其具备以下缺点：
 1. 单点故障，稳定性差
 2. 通信容量不足
 3. 远距离的集中式数据库，时延长
 4. 维护困难

 因此，现代 DNS 服务采用分布式设计

 ##### 分布式、层次数据库
 分层设计：
 1. 根 DNS 服务器
 2. 顶级域（TLD）DNS 服务器
 3. 权威 DNS 服务器
 树型结构，树型查找
 另有**本地 DNS 服务器**，三层 DNS 服务器之间并不直接通信，而是由本地 DNS 服务器逐层询问，直至权威 DNS 服务器响应目标地址
 递归查询 与 迭代查询
 - 区别：查询信息是否直接返回原主机，查询层次是不是只有一层

 ##### DNS 缓存
 在 DNS 服务器中 缓存 先前查询的记录结果，一定时间后丢弃缓存
 缓存的存在使得根服务器常常被绕过查询，减少了报文次数

### 2.4.3 DNS 记录和报文
 资源记录（RR Resource Record）提供主机名到 IP 地址的映射，存储在 DNS 服务器上
 RR 是一个四元组 `(Name, Value, Type, TTL)`
 - `TTL` 记录在缓存中保存的时间，超时则删除
 - `Name` 和 `Value` 的语义取决于 `Type` 的值
	 - `Type` 的值域为 `{A, NS, CNAME, MX}`，决定映射类型
	 - 对应映射类型为 `{主机-IP, 域DNS-权威DNS, 别名-规范名, 邮件服务器别名-规范名}`
 
 #### DNS 报文
 首部-问题-回答-权威-附加信息

 #### 在 DNS 数据库中插入记录
 向[[注册登记机构]]申请注册域名，提供 权威 DNS 服务器名字 与 IP 地址，机构会将对应的 RR 插入 TLD DNS 服务器 的数据库

*DNS 攻击实例* ：
- DDoS 攻击：大量的无效请求攻击 TLD DNS
- 伪造：截取 DOS 服务器的回答，伪装成 DNS 服务器，响应错误的报文，这会将用户重定向至攻击者的网站

## 2.5 [[P2P]]文件分发

P2P 采用对等连接，因此减少了对基础设施服务器的依赖

### P2P 体系结构的扩展性

随着用户数量增加，P2P 结构的文件分发能力也会增强，使得 P2P 具有自扩展性

下面计算 P2P 模型的传输效率
假设由一个服务器和 $N$ 个客户端组成 P2P 体系，分发一个 $F$ bit 的文件

记上载速率为 $u$ ，下载速率为 $d$

服务器将文件上传至因特网的时间至少为 $D_1 = \frac{F}{u_{server}}$
客户端从因特网下载文件的时间至少为 $D_2 = \frac{F}{d_{min}}$
系统的总上载能力为 $u_{total} = u_{server} + \sum\limits^{N}_{i = 1}u_i$ 
传输所有 $N$ 份文件的总上载时间至少为 $D_3 = \frac{NF}{u_{total}}$

因此 $D_{P2P} = max\{D_1, D_2, D_3\}$ 

### [[BitTorrent]]

一个具备若干用户的对等集合称为 洪流[[torrent]]
对等方之间彼此传递等长度的文件块[[chunk]]
每个 torrect 具备一个基础设施节点，称为追踪器[[tracker]]，负责记录与注册 torrent 中的用户

工作过程：
1. 当 Alice 加入 torrent 时，tracker 为其提供若干个对等方的 TCP 连接，Alice 将和他们成为邻居，邻居列表随时间而波动
2. Alice 询问邻居具有的 chunk，并请求下载自己缺失的 chunk
3. 同时 Alice 响应其他邻居的下载请求，优先分发**最稀缺**的 chunk
4. Alice 的响应策略是：向四位**上传速度最高**的邻居分发文件，同时每隔一段时间**随机**选中第五位邻居 Bob 进行分发
	- 这种激励机制称为 *tit-for-tat* ，强迫用户 Bob 提供自己的上载资源，否则将 Bob 永远不会处在 Alice 分发文件的黄金*五人*之内
	- 已经证明 tit-for-tat 可被人为回避

## 2.6 视频流和内容分发网

### 2.6.1 因特网视频
视频以比特传输，可以压缩至不同的比特率
视频传输要求稳定的平均端到端吞吐量
### 2.6.2 HTTP 流和 DASH
HTTP 中，视频以 URL 文件形式传输至客户缓存中，当客户端缓存达到一定界限，就开始播放视频，同时缓存后续的视频帧

但 HTTP 对应不同带宽的用户，需要传输不同比特率（清晰度）的视频文件，为此研发了 *经 HTTP 的动态适应性流（[[DASH]]）*，由客户端决定请求哪个版本的数据块

同时 DASH 动态地根据客户端的网络状况，来决定下一个要抓取的视频帧是什么版本的，如果抓了很多低清晰度画面，却发现还有大量带宽闲余，DASH 就会选择接下来切换到高清晰度
### 2.6.3 内容分发网

实际产品：YouTube

最简单的服务提供方式：部署单一的巨大数据中心
缺点：时延、不稳定、反复传输

因此利用**内容分发网（[[CDN]]）** 
- CDN 服务器部署在多个地理位置
- 专用 CDN 和第三方 CDN 均可

CDN 的服务器安置原则
1. 深入
	- 在全球各处的接入 ISP 中部署 CDN 集群
	- 目的为靠近端用户，采用高度分布式设计
2. 邀请做客
	- 在少量关键位置建造大集群，邀请 ISP 做客
	- 通常放置在 IXP 处
	- 维护成本低，但用户体验不如深入部署

CDN 的传输策略类似于 Web 缓存，当用户请求时才将视频从远程拉入本地副本，一段时间后删除副本

#### CDN 操作
当用户发出视频请求时，CDN 截获报文，决定使用哪个 CDN 集群，并重定向至新的服务器

以使用 DNS 截取的 CDN 为例，工作过程如下：
1. 用户访问 Web 网页
2. 用户点击视频连接，发送 DNS 请求
3. 本地 DNS 服务器接收到请求，注意到这是一个视频相关，返回一个 CDN 域的主机名（而不是目标的 IP 地址），移交 DNS 请求
4. CDN 集群给出用户应该连接的服务器地址，返回给本地 DNS 服务器（LDNS）
5. [[LDNS]] 返回 CDN 给出的 IP 地址
6. 客户端与收到的 IP 建立 直接的 TCP 连接，开始视频服务

#### 集群选择策略
最简单的策略是：选择地理位置最近的集群，但**时延不一定最短**
部分 CDN 进行对集群的时延周期性测量，缺点是浪费一定 LDNS 资源

### 2.6.4 学习案例： Netflix 和 YouTube
#### Netflix
两个部件：亚马逊云、专用 CDN 基础设施
- Web网站控制用户相关数据，运行在亚马逊云上
- 此外，亚马逊云处理以下功能
	- 内容摄取
	- 内容处理
	- 向 CDN 上载版本
- Netflix CDN 部署在 ISP 和 IXP 上，并专门用于 视频流 的播放，简化 CDN 设计
- 同时 CDN 采用推高速缓存，空闲时将内容推入下层服务器
Netflix Web 向用户提供视频 URL 的相关信息，然后客户端向 CDN 请求数据即可

#### YouTube
采用拉高速缓存和 DNS 重定向，以平衡集群负载
应用 HTTP 流，没有适应性流，要求用户人工选择版本
数据处理全部发生在谷歌数据中心
## 2.7 [[套接字]]编程：生成网络应用
### 2.7.1 UDP 套接字编程
下面代码实现四步流程：
1. 客户键入字符，发送给服务器
2. 服务器接收到信息，把字母转换成大写
3. 服务器把新数据返回给客户端
4. 客户端收到数据，并显示到监视器

#### UDPClient.py
```py
from socket import *

serverName = 'hostname'
serverPort = 12000

clinetSocket = socket(AF_INET, SOCK_DGRAM) # 使用 IPV4, UDP

message = raw_input('Input lowercase sectence:')

clientSocket.sendto(message.encode(), (serverName, serverPort))

modifiedMessage, serverAddress = clientSocket.recvfrom(2048) 
# 2048 为读入的缓存长度

print(modifiedMessafe.decode()) 
# encode()/decode() 用于把数据转换成字节流

clientSocket.close()
```


#### UDPServer.py
```py
from socket import *

serverPort = 12000
serverSocket = socket(AF_INET, SOCK_DGRAM);
serverSocket.bind('', serverPort)

print("The server is ready to receive")

while True :
	message, clientAddress = serverSocket.recvfrom(2048)
	modifiedMessage = message.decode().upper()
	serverSocket.sendto(modifiedMessage.encode(), clientAddress)
```

### 2.7.2 TCP 套接字编程
与前一节相同的流程
需要三次握手，前两次建立连接后，服务端另开一个新的套接字端口，用于数据传输
前两次握手均通过专用的*欢迎端口* 
#### TCPClient.py
```py
from socket import *

serverName = 'hostname'
serverPort = 12000

clientSocket = socket(AF_INET, SOCK_STREAM) # IPV4, TCP

clientSocket.connect((serverName, serverPort))

sentence = raw_input('input lowercase sentence:')

clientSocket.send(sentence.encode())

modifiedSentence = clientSocket.recv(1024)

print('From Server: ', modifiedSentence.decode())

clientSocket.clode()
```


#### TCPServer.py
```py
from socket import *

serverPort = 12000

serverSocket = socket(AN_INFT, SOCK_STREAM)

serverSocket.bind(('', serverPort))

serverSocket.listen(1) # 第一次握手

print('The server is ready to receive')

while True:
	connectionSocket, addr = serverSocket.accept()
	sentence = connectionSocket.recv(1024).decode()
	capitalizeSentence = sentence.upper()
	connectionSocket.send(capitalizedSentence.encode())
	connectionSocket.close()
```