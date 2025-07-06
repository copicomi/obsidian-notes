# 实验一

1. 在实验报告中写出网络常用命令的操作过程及效果，分析并总结实验中遇到的问题，写出实验体会。

2. 在实验报告中写出DNS层次查询的实现过程、利用SMTP协议实现接发邮件的过程、回答捕包实验中所提问题。分析并总结实验中遇到的问题，写出实验体会。

(1)列出在第7步中分组列表子窗口所显示的所有协议类型。

(2)从发出HTTP GET报文到接收到HTTP OK响应报文共需要多长时间？（在默认的情况下，分组列表窗口中Time列的值是从Ethereal开始追踪到分组被俘获的总的时间数，以秒为单位。若要按time-of-day格式显示Time列的值，需选择View下拉菜单，再选择Time Display Format，然后选择Time-of-day。）

(3)你主机的IP地址是什么？你所访问的主页所在服务器的IP地址是什么？

(4)写出两个第9步所显示的HTTP报文头部行信息。

---
## **实验内容**

网络常用命令的使用及DNS层次查询、SMTP协议分析

---

## **实验目的**

1. 掌握PING、Tracert、Netstat、IPCONFIG、ARP等网络命令的使用方法及参数含义。
    
2. 通过DNS层次查询理解域名解析过程，掌握NSLOOKUP命令的使用。
    
3. 分析SMTP协议的工作流程，通过Telnet手动发送邮件。
    
4. 使用Ethereal（Wireshark）捕包工具分析网络协议交互过程。
    

---

## **实验预备知识**

1. **DNS分层结构**：根域名服务器→顶级域（如.com）→权威域名服务器→本地DNS服务器。
    
2. **SMTP协议**：基于TCP 25端口，命令为明文（如`HELO`、`MAIL FROM`、`RCPT TO`、`DATA`）。
    
3. **捕包工具**：Ethereal/Wireshark可捕获并解析各层协议（如以太网帧、IP、TCP、HTTP）。

---

## 实验过程描述

1. 网络常用命令：
	- **Ping**：
		- 命令： ***ping baidu.com -n 3***
		- 功能：向 baidu.com 发送 3 个数据包
		- **结果**：成功连通，平均用时为 17 ms
	- Tracert：
		- 命令： tracert -d baidu.com
		- 功能：追踪到 baidu.com 的路径，不解析主机名
		- 结果：经 17 跳到达目的地，第 1 跳的 IP 地址为 10.241.0.1，第 4~7 跳请求超时
		- 分析：超时原因可能是 数据包跨过 ISP 进行传输
	- Netstat
		- 命令： netstat -no
		- 功能：显示本地的活动连接，以数字形式显示地址和端口号，显示进程 pid
		- 结果：得到一张表头为“协议-本地地址-外部地址-状态-PID”的连接状态表，注意到 进程 1820 使用 TCP 协议，占用 443 端口（用于 HTTPS 服务）
	- Arp
		- 命令： arp -a
		- 功能：显示本地所有接口的 ARP 缓存表
		- 结果：得到一张缓存表，查询到 10.241.0.1 的物理地址为 58-69-6c-a6-8f-09，类型为动态
	- IPConfig
		- 命令： ipconfig /release && ipconfig /renew
		- 功能：释放 IP 并刷新 DHCP 
		- 结果：使用 'ipconfig -all' 命令查看 DHCP 的租约时间，发现已经得到更新
---
2. DNS 层次查询
	- 先查询根服务器名称
		- 命令：nslookup -qt=NS .
		- 结果：获得所有根服务器的名字
	- 再逐层查询 baidu.com
		- 命令：nslookup -qt=NS <目标域名> <指定DNS服务器域名>
		- 结果：逐级向下查询，依次获得 顶级域名 DNS 服务器（.com）、权威 DNS 服务器（ns1.baidu.com)、目标域名的 相关信息
---
3. 利用 TELNET 进行 SMTP 邮件发送
	- telnet smtp.163.com 25
	- HELO <本地名>
	- AUTH LOGIN
	- <base64用户名>
	- <base64密码>
	- MAIL FROM: <发送者邮箱>
	- RCPT TO: <收信者邮箱>
	- DATA
	- From: <发送者邮箱>
	- To: <收信者邮箱>
	- Subject: Test
	- This is a test email.
	- .
	- QUIT
---
4. 捕包实验（Wireshark）
	- 选择过滤 http
	- 连接 http://cs144.keithw.org/lab0/
	- 捕获到 GET 报文和 200 OK 报文
---
## 实验结果

1. 熟悉了基本网络命令的使用，查看了本机的网络环境
2. 成功使用 nslookup 查询具体域名，解析了 DNS 的层次结构
3. 用 telnet 成功进行了 STMP 邮件发送，采用 163 邮箱做实验，收发邮箱双方均检查到有测试邮件
4. 捕包实验问题回答：(用连接 http://cs144.keithw.org/lab0/ 做实验)
	1. 类型：TCP、HTTP、DNS、ARP
	2. 测得 收到 GET 报文 和 200 OK 报文 的 间隔时间为 0.2468s
	3. 主机 IP 为 10.241.203.158，目标服务器 IP 为 104.196.238.229
	4. GET 和 OK 报文的头部信息为 【TO ADD：本地的运行截图】 

---
## 遇到的问题和解决方案

1. 发送 STMP 邮件时，输入命令过慢，导致超时断开连接
	- 解决方案：提前写好命令，复制粘贴，或使用管道输入
2. 连接 baidu.com 时，总是收不到 HTTP 包
	- 原因：chrome浏览器默认使用 HTTPS 连接
	- 解决方法：尝试从浏览器的高级设置中，强制使用 http 访问 baidu.com，多次尝试仍不成功，换用http://cs144.keithw.org/lab0/ 做实验
