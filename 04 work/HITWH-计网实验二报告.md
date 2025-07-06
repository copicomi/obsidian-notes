l **利用ethereal****分析协议HTTP、FTP和DNS**

一、 **实验目的**

1、 分析HTTP协议

2、 分析DNS协议

二、 **实验环境**

与因特网连接的计算机网络系统；主机操作系统为windows；Ethereal、IE等软件。

三、 **实验步骤**

1、 HTTP GET/response交互

首先通过下载一个非常简单的HTML文件（该文件非常短，并且不嵌入任何对象）。

（1） 启动Web browser。

（2） 启动Ethereal分组嗅探器。在窗口的显示过滤说明处输入“http”，分组列表子窗口中将只显示所俘获到的HTTP报文。

（3） 一分钟以后，开始Ethereal分组俘获。

（4） 在打开的Web browser窗口中输入一下地址（浏览器中将显示一个只有一行文字的非常简单的HTML文件）：

http://gaia.cs.umass.edu/ethereal-labs/HTTP-ethereal-file1.html

（5） 停止分组俘获。

窗口如图1所示。根据俘获窗口内容，回答“四、实验报告内容”中的1-6题。

![](file:///C:\Users\lyx\AppData\Local\Temp\ksohtml19948\wps1.png) 

图1分组俘获窗口

2、 HTTP 条件GET/response交互

（1） 启动浏览器，清空浏览器的缓存（在浏览器中，选择“工具”菜单中的“Internet选项”命令，在出现的对话框中，选择“删除文件”）。

（2） 启动Ethereal分组俘获器。开始Ethereal分组俘获。

（3） 在浏览器的地址栏中输入以下URL:  http://gaia.cs.umass.edu/ethereal-labs/HTTP-ethereal-file2.html,你的浏览器中将显示一个具有五行的非常简单的HTML文件。

（4） 在你的浏览器中重新输入相同的URL或单击浏览器中的“刷新”按钮。

（5） 停止Ethereal分组俘获，在显示过滤筛选说明处输入“http”,分组列表子窗口中将只显示所俘获到的HTTP报文。

根据操作回答“四、实验报告内容”中的7-10题。

3、 获取长文件

（1） 启动浏览器，将浏览器的缓存清空。

（2） 启动Ethereal分组俘获器。开始Ethereal分组俘获。

（3） 在浏览器的地址栏中输入以下URL: http://gaia.cs.umass.edu/ethereal-labs/HTTP-ethereal-file3.html，浏览器将显示一个相当大的美国权力法案。

（4） 停止Ethereal分组俘获，在显示过滤筛选说明处输入“http”,分组列表子窗口中将只显示所俘获到的HTTP报文。

根据操作回答“四、实验报告内容”中的11-13题。

4、 嵌有对象的HTML文档

（1） 启动浏览器，将浏览器的缓存清空。

（2） 启动Ethereal分组俘获器。开始Ethereal分组俘获。

（3） 在浏览器的地址栏中输入以下URL: http://gaia.cs.umass.edu/ethereal-labs/HTTP-ethereal-file4.html，浏览器将显示一个具有两个图片的短HTTP文件

（4） 停止Ethereal分组俘获，在显示过滤筛选说明处输入“http”,分组列表子窗口中将只显示所俘获到的HTTP报文。

根据操作回答“四、实验报告内容”中的14-15题。

5、 HTTP认证

（1） 启动浏览器，将浏览器的缓存清空。

（2） 启动Ethereal分组俘获器。开始Ethereal分组俘获。

（3） 在浏览器的地址栏中输入以下URL: https://gaia.cs.umass.edu/ethereal-labs/protected_pages/HTTP-ethereal-file5.html，浏览器将显示一个HTTP文件，输入所需要的用户名和密码(用户名：eth-students,密码:network)。

（4） 停止Ethereal分组俘获，在显示过滤筛选说明处输入“http”,分组列表子窗口中将只显示所俘获到的HTTP报文。

根据操作回答“四、实验报告内容”中的16-17题。

6、 跟踪DNS

nslookup工具允许运行该工具的主机向指定的DNS服务器查询某个DNS记录。如果没有指明DNS服务器，nslookup将把查询请求发向默认的DNS服务器。其命令的一般格式是：

nslookup –option1 –option2 host-to-find dns-server

ipconfig命令用来显示你当前的TCP/IP信息，包括：你的地址、DNS服务器的地址、适配器的类型等信息。如果，要显示与主机相关的信息用命令：

ipconfig/all

如果查看DNS缓存中的记录用命令：

ipconfig/displaydns

要清空DNS缓存，用命令：

ipconfig /flushdns

**运行以上命令需要进入MSDOS环境。**

（1） 利用ipconfig命令清空你的主机上的DNS缓存。

（2） 启动浏览器，将浏览器的缓存清空。

（3） 启动Ethereal分组俘获器，在显示过滤筛选说明处输入“ip.addr==your_IP_address”(如：ip.addr==10.17.7.23)，过滤器将会删除所有目的地址和源地址都与指定IP地址不同的分组。

（4） 开始Ethereal分组俘获。

（5） 在浏览器的地址栏中输入：[http://www.ietf.org](http://www.ietf.org)

（6） 停止分组俘获。

根据操作回答“四、实验报告内容”中的18-24题。

（7） 开始Ethereal分组俘获。

（8） 在www.mit.edu上进行nslookup（即执行命令：nslookup www.mit.edu）。

（9） 停止分组俘获。

根据操作回答“四、实验报告内容”中的25-28题。

（10） 重复上面的实验，只是将命令替换为：nslookup –type=NS mit.edu

根据操作回答“四、实验报告内容”中的29-31题。

（11） 重复上面的实验，只是将命令替换为：nslookup [www.aiit.or.kr](http://www.aiit.or.kr) bitsy.mit.edu

根据操作回答“四、实验报告内容”中的32-34题。

四、 **实验报告内容(全部问题必须实际操作并思考，可以选做8道题)**

在实验的基础上，回答以下问题：

（1）你的浏览器运行的是HTTP1.0，还是HTTP1.1？你所访问的服务器所运行的HTTP版本号是多少？

（2）你的浏览器向服务器指出它能接收何种语言版本的对象？

（3）你的计算机的IP地址是多少？服务器gaia.cs.umass.edu的IP地址是多少？

（4）从服务器向你的浏览器返回的状态代码是多少？

（5）你从服务器上所获取的HTML文件的最后修改时间是多少？

（6）返回到你的浏览器的内容一共多少字节？

（7）分析你的浏览器向服务器发出的第一个HTTP GET请求的内容，在该请求报文中，是否有一行是：IF-MODIFIED-SINCE？

（8）分析服务器响应报文的内容，服务器是否明确返回了文件的内容？如何获知？

（9）分析你的浏览器向服务器发出的第二个“HTTP GET”请求，在该请求报文中是否有一行是：IF-MODIFIED-SINCE？如果有，在该首部行后面跟着的信息是什么？

（10）服务器对第二个HTTP GET请求的响应中的HTTP状态代码是多少？服务器是否明确返回了文件的内容？请解释。

（11）你的浏览器一共发出了多少个HTTP GET请求？

（12）承载这一个HTTP响应报文一共需要多少个data-containing TCP报文段？

（13）与这个HTTP GET请求相对应的响应报文的状态代码和状态短语是什么？

（14）你的浏览器一共发出了多少个HTTP GET请求？这些请求被发送到的目的地的IP地址是多少？

（15）浏览器在下载这两个图片时，是串行下载还是并行下载？请解释。

（16）对于浏览器发出的最初的HTTP GET请求，服务器的响应是什么(状态代码和状态短语)?

（17）当浏览器发出第二个HTTP GET请求时，在HTTP GET报文中包含了哪些新的字段？

（18）定位到DNS查询报文和查询响应报文，这两种报文的发送是基于UDP还是基于TCP的？

（19）DNS查询报文的目的端口号是多少？DNS查询响应报文的源端口号是多少？

（20）DNS查询报文发送的目的地的IP地址是多少？利用ipconfig命令（ipconfig/all）决定你主机的本地DNS服务器的IP地址。这两个地指相同吗？

（21）检查DNS查询报文，它是哪一类型的DNS查询？该查询报文中包含“answers”吗？

（22）检查DNS查询响应报文，其中提供了多少个“answers”？每个answers包含哪些内容？

（23）考虑一下你的主机发送的subsequent(并发)TCP SYN分组， SYN分组的目的IP地址是否与在DNS查询响应报文中提供的某个IP地址相对应？

（24）打开的WEB页中包含图片，在获取每一个图片之前，你的主机发出新的DNS查询了吗？

（25）DNS查询报文的目的端口号是多少？DNS查询响应报文的源端口号是多少？

（26）DNS查询报文发送的目的地的IP地址是多少？这个地址是你的默认本地DNS服务器的地址吗？

（27）检查DNS查询报文，它是哪一类型的DNS查询？该查询报文中包含“answers”吗？

（28）检查DNS查询响应报文，其中提供了多少个“answers”？每个answers包含哪些内容？

（29）DNS查询报文发送的目的地的IP地址是多少？这个地址是你的默认本地DNS服务器的地址吗？

（30）检查DNS查询报文，它是哪一类型的DNS查询？该查询报文中包含“answers”吗？

（31）检查DNS查询响应报文，其中响应报文提供了哪些MIT名称服务器？响应报文提供这些MIT名称服务器的IP地址了吗？

（32）DNS查询报文发送的目的地的IP地址是多少？这个地址是你的默认本地DNS服务器的地址吗？如果不是，这个IP地址相当于什么？

（33）检查DNS查询报文，它是哪一类型的DNS查询？该查询报文中包含“answers”吗？

（34）检查DNS查询响应报文，其中提供了多少个“answers”？每个answers包含哪些内容？

---
# 实验内容

1. 利用ethereal分别对TCP套接字的实现及UDP套接字的实现捕包分析
2. 利用ethereal分析协议HTTP、FTP和DNS