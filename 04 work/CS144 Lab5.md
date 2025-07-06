实现网络接口

---

`send_datagram( IP_datagram, next_IP)`

发送 IP 数据报 到 下一跳的 IP 地址

1. 如果知道下一跳的 MAC 地址，封装为 以太网帧 后直接发送
2. 否则，广播 ARP 请求，将 IP 数据报 加入队列，收到回复后发送

- `5s` 内不再发送对同一个 IP 地址的 ARP 请求

---
`recv_frame( frame )`

接收以太网帧

1. 如果是 IPv4，解析为 IP 数据报，加入 receive 队列
2. 如果是 ARP 
	- 解析为 ARP 请求，继续广播，或者回复询问
	- 记住发送方的 IP 地址与 MAC 映射，记忆 `30s`

---
`tick()`

模拟时间流逝，控制信息的时效性

---

#### 数据结构设计

1. `map < ip_adr, {mac_adr, tick_time} >` 
	- 记录 地址映射，记忆 30s 
2. `queue < ip_data >` receive
	- 记录 ip 数据报 队列
3. `map < ip_adr, tick_time >`
	- 记录 发送过的 arp 请求，记忆 5 s
4. `queue < {ip_datagram, next_hop} >`
	或 `map < next_hop, queue < ip_datagram > >`
	- 记录 正在等待 ARP 回复的 IP 数据报
5. `_arp_clock`
	- 用于 ARP 处理过期信息的 时钟

---

#### 算法设计

`send_datagram()`
- 检查 ARP 表，查询 `next_hop` 对应的 MAC 地址
	- 查询成功：
		- 创建一个新的以太网帧
		- 调用 `transmit()` 直接发送
	- 查询失败：
		- 将该 IP 数据报，加入等待队列 A
		- 构造 ARP 广播帧，询问 `next_hop` 的 MAC
			- 如果 5s 内，访问过同一 IP
				- 即 `next_hop` 处在 `map` 表中
				- 那么就不发送 ARP 请求广播
			- 否则，发送 ARP 请求广播
				- 并将 `next_hop` 插入 `map` 表
---
`recv_frame()`
- 解析以太网帧，忽略其他类型、或不以本机为目标的帧
- 若为 `IPv4` 类型
	- 将数据解析为 `IP_datagram`
	- 若成功解析，则将其加入 收到队列 B
- 若为 `ARP` 类型
	- 将数据解析为 `ARPMessage`
	- 若成功，则将发送方的 IP 和 MAC 映射加入 `map` 表
		- 若为 `ARP` 广播
			- 询问对象为本机，直接发送回复
			- 否则，继续传递广播
		- 若为 `ARP` 回复
			- 不作反应，因之前已将新映射加入 `map` 表
			- 检查 `queue` 中是否有 `IP_datagram` 正在等待该 回复
				- 如果有，调用 `transmit()` 发送该信息
---
`tick()`
- 更新 `_arp_clock`
- 检查两个 `map` ，移除超时的对象

---

> ：如何发送 以太网帧？
> 调用 `transmit()`

> :如何解析 以太网帧？

> :如何构造 以太网帧？

> :如何检测 ARP 信息