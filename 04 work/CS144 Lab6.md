实现路由器转发逻辑

---
`void add_route( route_prefix, prefix_length, next_hop, interface_num)`

添加路由表项

路由前缀、前缀长度、下一跳IP，端口号


---

`void route()`

搜索路由表，最长前缀匹配 路由前缀

匹配失败，直接丢弃 IP 数据报

递减 TTL，为零或达到零，丢弃

通过接口转发数据报
next_hop 为空时，直接发送到 IP 数据报的 dst_ip_address

---
## 数据结构设计

`map< IP_Address_prefix, {next_hop, interface_num} >`
- 存储路由表，对每个前缀长度建一个 `map`，共 `32` 个

---

