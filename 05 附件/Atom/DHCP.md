**动态主机配置协议**

DHCP 允许主机从连接的子网中自动获取一个地址，**或临时或固定**，同时让主机得知必要的子网环境信息

这种自动化的方式，省去了手工配置的工作，更符合有大量主机频繁加入和退出的网络环境

每个子网都应具有一台 DHCP服务器，或 [[DHCP 中继代理]]

DHCP 采用 client-server 模型

DHCP 的工作过程：
1.  DHCP 服务器发现
	 - 起初，主机 C 并不知道自己所处的网络环境，必须通过广播手段，发送 *DHCP 发现报文（discover msg）*，使用 *广播目的地址 255.255.255.255，主机源地址 0.0.0.0 *
2.  DHCP 服务器提供
	 - 当 DHCP 服务器 S 收到 discover msg 时，同样开始广播 *DHCP 提供报文（offer msg）*
	 - offer msg 包含向 主机 C 提供的 地址、掩码、**IP地址租用期** 
3.  DHCP 请求
	 - 主机 C 可能收到多个 offer msg，此时择优发送 *DHCP 请求报文 （request msg）*
	 - request msg 包含回显参数
4.  DHCP ACK
	- 服务器 S 响应 ACK msg
	- 当 主机 C 收到 ACK msg，服务结束

