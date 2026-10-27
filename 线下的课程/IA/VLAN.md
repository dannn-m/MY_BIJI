# VLAN 的类型
1. 基于端口
2. 基于 mac
3. 基于协议
# 端口类型
- 交换机与终端计算机直接相连的链路成为 access
- 交换机之间直接相连的链路称为 trunk
- 每一几交换机端口（无论 access 还是 trunk）都应该配置一个 PVID，到达这个端口的 Untagged 帧将一律被交换机划分到 PVID 所代指的 VLAN
- （1）Access 端口
	- 当 从链路上接收到一个帧后
		- untag，打上 pvid 的 tag
		- tag，不同就丢弃
	- 当收到一个 ta
	- 当交换机上其他端口进入 access 口，交换机会检查这个 tag 中的 VID 与此 PVID 相同则转发
- （2）Trunk 端口
	- 从链路上收到一个 untagged 交换机会添加 pvid 的 tag，然后检查是否在允许通过列表再转发
	- 链路上收到一个 tagged，检查是否在允许通过列表
	- 当本交换机其他口到达一个 trunk 口，当这个帧在允许列表，且 VID 与 PVID 相同，交换机会对这个 tag 帧进行剥离，并发送
	- tag 帧到从本交换机此 trunk 口，在允许列表中，VID 与 PVID 不同，交换机会直接将它发出去而不进行剥离
- （3）Hybrid 端口
	- 除了要配置 PVID 外还要配置 untagged 列表和 tagged 列表