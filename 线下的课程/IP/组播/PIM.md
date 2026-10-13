- 协议无关组播
- 用于构建组播分发树

- 工作在组播转发网络的协议：PIM、MSDP、MBGP
	- PIM（协议无关组播）：生成 as 域内的组播分法树
	- MSDP(组播源发现协议)：帮助生成 AS 域间的组播分发树
	- MBGP（组播 bgp）：帮助跨域组播流进行 RPF 校验
#  介绍
- 直接利用单播路由表的路由信息，对组播报文执行 RFP 检查，检查后创建组播转发表项
- 是实际网络中有两种模式
	- PIM-DM(PIM-Dense-Mode，PIM 密集模式)
	- PIM-SM(PIM-Sparse-Mode，PIM 稀疏模式)，又可以分为
		- PIM-SM(ASM)：为任意源组播建立组播分发树
		- PIM-SM(SSM)：为指定源建立组播分发树