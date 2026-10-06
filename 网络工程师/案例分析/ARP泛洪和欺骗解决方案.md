# 泛洪
- 原因：主机中毒、环路
方案：
- 在接口下配置 arp 限制表项
	- arp－limit
- 配置针对源 ip 地址的 arp 报文速率抑制功能
	- arp speed－limit source－ip x.x.x.x maxmum 10
- 攻击源 mac 过滤
	- acl 
	- rule permit l 2－protocol arp source－mac x vlan－id x
	- q
	- cpu defend policy 1
	- blacklist 1 acl x
- 用流量分类器使用 qos 的方式进行限速
	- traffic behavior policy 1
	- car cir 32
# 欺骗
- 现象：局域网内时通时断，网络设备会拖管，网关设备打印大量地址冲突
