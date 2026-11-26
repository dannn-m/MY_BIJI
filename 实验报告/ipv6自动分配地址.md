# 静态
# 动态
> 状态指是否有中心服务器记录并维护分配信息
> 是否采用无状态方式有 RA 报文中的 m 和 o 标志位决定

> m 决定地址是否采用无状态方式，缺省为 0（无状态）
> o 决定地址 以外其他信息是否采用无状态方式，缺省为 0（无状态）

## 常见的搭配
| m   | o   | 含义                       |
| --- | --- | ------------------------ |
| 0   | 0   | 无状态自动分配地址 SLAAC，所有信息均无状态 |
| 0   | 1   | 无状态 DHCP                 |
| 1   | 1   | 有状态 DHCP                 |
- SLAAC
	- 在路由器接口视图开启 undo ipv 6 nd ra halt（解除端口抑制周期发送 ra）
	- 在接受无状态自动地址分配的端口配置 ipv add auto linklocal （global）
- 无状态 dhcp
	- 全局开启 DHCP 功能
	- dhcpv pool *name* 
		- exclude ..
		- dns
	- 接口视图
		- dhcp sever *name*
		- ipv nd other-flag 将 o 位置位 
- 有状态（只列出了对于无状态 dhcp 的区别）
	- 在 pool 视图下
		- address prefix xxxx（分配的前缀列表）
	- 接口s