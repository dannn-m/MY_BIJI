




# ospf
seriol的串口的带宽很低默认1562
ospf的nbma的邻居需要手动创建
peer IP地址
![[Pasted image 20260805141546.png]]
对于主机可以设置静默接口，不收发路由器的hello包

---
# 路由引入
动态路由双向引入
静态路由单项引入


# 路由汇总
静态汇总是自己配的
动态汇总是别人给的
ospf是基于区域内的链路状态路由协议，区域内的链路状态应该被毫无保留的展现而不应该被隐藏


# lsa



# 小一点的知识点
## 端口映射
### 普通映射
### 流映射

---
## BFD
双向转发检测
故障检测技术

---
## 端口安全

接入层交换机一个端口一个终端一个mac
非接入交换机一个端口多个设备多个mac
一般的端口安全：
限制mac表的学习数量
有几个模式：普通丢弃，丢弃并上报、


---
## lldp



---
## muxvlan
- vlan可以配置主从vlan
- 从vl间分为互通型子vl（vl内主机可互通）与隔离型子vl（反之）
- 子vl间不通，任意子vl都可与主vl通信
~~~
2 创建VALN
[Huawei]vlan batch 10 20 100
3 将接口加入到VLAN
interface GigabitEthernet0/0/1
 port link-type access
 port default vlan 10
4 Mux VLAN配置
[Huawei-vlan100]mux-vlan
[Huawei-vlan100]subordinate separate 20 
[Huawei-vlan100]subordinate group 10
5 在接口上启用Mux VLAN功能
[Huawei-GigabitEthernet0/0/1]port mux-vlan enable
~~~
---
## supervlan
- 基于同网段相同但是vl不同的需求，需要设置一个supervlan
- 在supervl中视图中将子vl划入其中，开启arp 路由间代理
- 给supervl的三层口配置一个ip。
---

## 动态vlan
- 可以选择配置基于mac、ip、协议、策略
- 策略：
	- 动静结合
	- 两个方案：ip+mac +端口

---

## GVRP
- garp：通用属性注册协议（本身是一个协议框架）
	- gvrp：通用vlan注册协议
	- gmrp（multicast registration）
作用：动态学习vl不需要在在每个交换机创建
 dis gvrp status
 dis gvrp int g0/0/0

---
## qinq
用于将私网的vl包再次封装一个vl，qinq即802.1q里面封装802.1q
### qinqport
- 普通的没有灵活的自定义封装私网vlan
### qinq stacking
- 可以自定义封装关系如：10封装到100中  20封装到200中 

---

## vlanmap



---

## DHCP


---
## arp代理


---



## VRRP
越低越优先，理论0-255 实际0-254
组播报文目的224.0.0.18
vrrp定时器


--- 
## nqa
网络质量分析
bfd是业务技术


---
## 前缀列表
ipprefix 

---
## isis
- isis的区域边界是整个路由器ospf的区域边界在router接口
- l2路由器所在的范围，称之为骨干
- L1/2路由器等同于ospf中abr的效果
- 连续的L1所形成的范围称之为区域