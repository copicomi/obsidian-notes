多协议标签交换，Multiprotocol Label Switching

---
---

### 多协议标签交换

MPLS 引入固定长度标签，允许路由器根据该标签进行转发（而不是仅仅以 IP 地址为标准），该技术与 IP 协同工作

[[MPLS]] 在链路层帧 添加一个 MPLS 首部，该帧只能通过 MPLS 使能的路由器（称为*标签交换路由器*）交换

---

MPLS 的交换过程，不需考虑 IP 地址，加快了运行速度

但 MPLS 的优势不仅在于效率，还在于其潜在的 **流量管理能力**

通过对 MPLS 标签的配置，我们能够做到 **定制化** 数据转发的 路径，从而 **控制流量**

MPLS 也可以用于 实现 [[VPN]]，不采用路径最近的行为，而根据 MPLS 标签的配置，“人为地”选择绕远路，通过中间站 VPN 访问因特网

---
