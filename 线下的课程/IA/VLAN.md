# VLAN 的类型
1. 基于端口
2. 基于 mac
3. 基于协议
# 端口类型
- 交换机与终端计算机直接相连的链路成为 access
- 交换机之间直接相连的链路称为 trunk
- 每一几交换机端口（无论 access 还是 trunk）都应该配置一个 PVID，到达这个端口的 Untagged 帧将一律被交换机划分到 PVID 所代指的 VLAN
- （1）Access 端口
	- 当 Access 端口从链路上接收到一个 Untagged 帧后，交换机会在这个帧中添加上 VID 为 PVID 的 tag
	- 当收到一个 tagged 帧，交换机会检查这个帧的 Tag 中的 VID 是否与 PVID 相同，不同则丢弃
	- 当交换机上其他端口进入 access 口，交换机会检查这个 tag 中的 VID 与此 PVID