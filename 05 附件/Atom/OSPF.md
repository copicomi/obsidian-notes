

---
---
#### 因特网中自治系统内部的路由选择：[[OSPF]]

>为什么选择 OSPF？
>- 规模过大的网络，路由复杂度过高，需要分块
>- ISP 存在自行管理内网的需求

这种情况通过把路由器组织进 *[[自治系统]]（AS）* 来解决

每个 AS 内部的路由器，信息共享，采用同样的路由选择算法，称为 *[[自治系统内部路由选择协议]]*

---
### 开放最短路优先（OSPF）

OSPF 是一种链路状态协议，使用洪泛链路状态信息和 Dijkstra 最低开销路径算法

每台路由器本地运行 Dijkstra 算法

链路开销 由 网络管理员 手工配置，具备灵活性

路由器定期向 全内网 **广播** 其 路由选择信息，增加算法健壮性

OSPF 的优点：
- 安全
- 允许使用多条相同开销的路径
- 对单播与多播路由选择的综合支持
- 支持在单个 AS 中的层次结构，分层配置路由器
---