- 直接可以参考 报文大全里面的 LSA
- 摘要几点：
# 一类 LSA

| Link Type                                                  | Link ID                        | Link Data                    |
| ---------------------------------------------------------- | ------------------------------ | ---------------------------- |
| Point-to-Point（P 2 P）：描述从本路由器到邻居路由器之间的点到点链路，属于拓扑信息         | 邻居路由器的 Router ID               | 宣告该 Router LSA 的路由器接口的 IP 地址 |
| TransNet：描述从本路由器到一个 Transit 网段（例如 MA 或者 NBMA 网段）的连接，属于拓扑信息 | DR 的接口 IP 地址                   | 宣告该 Router LSA 的路由器接口的 IP 地址 |
| StubNet：描述从本路由器到一个 Stub 网段（例如 Loopback 接口）的连接，属于网段信息       | 宣告该 Router LSA 的路由器接口的网络 IP 地址 | 该 Stub 网络的网络掩码               |
# 二类 LSA
- link state DR 的接口 IP 地址
- Network Mask：MA 网络的子网掩码
- Attached Router：连接到该 MA 网络的路由器的 router－id（与该 DR 建立了邻接关系的邻居的 router－id，以及 DR 自己的 router－id）