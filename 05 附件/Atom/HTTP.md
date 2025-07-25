HTTP定义了请求网页和传送页面的方式
HTTP 采用 TCP
HTTP 不保存来自客户端的状态，是一个[[无状态协议]]

---
#### 非持续连接的HTTP
工作过程
1. HTTP 客户端 向 目标服务器 发起 TCP 连接
2. 经 socket 发送请求报文，包含请求文件的路径
3. 服务端 经 socket 接收请求，封装请求对象，发送响应报文
4. 服务端通知 断开 TCP 连接
5. 客户端接收响应报文，TCP 连接关闭
6. 收到的 HTML 文本，可能包含其他存储在服务端的对象URL，对每个 URL 递归调用上述过程
---
### HTTP 报文格式

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
---
### HTTP/2
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

---

