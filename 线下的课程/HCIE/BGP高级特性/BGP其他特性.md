# ORF
- 出口路由过滤，利用 BGP 建立邻居时协商的 router－refresh 能力协商路由
- 当出口路由器在出方向无法在出方向做路由策略（距离限制）
	- 对出口路由器性能消耗，就需要在接受路由器配置 ORF
# BGP 对等体组
某些相同策略的对等体集合，对齐进行整体配置
- ibgp group
- ebgp group
	- **group** *name* e/i
	- peer adr group name
	- peer name as－num  100
		- 针对 ebgp 需要在创建邻居的时候指定 as 号，igp 不用写，与自身 as 号相同
# BGP 安全性
常见攻击 ：
	建立非法邻居
	非法 bgp 报文
解决办法：BGP 认证
- 