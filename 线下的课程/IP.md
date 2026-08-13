




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
- 连续的L1所形成的范围称之为区域（ospf末梢）
- 区域内的L1路由器只能和本区域连接不能跨区域
- 各类型的路由器只能和相邻的同类型建立邻居
- L1路由器只拥有本区域的路由

---
## bgp
- 预先规划as_path一般叠加自身编号
- 本地优先级作用于出去的流量默认值100
- med值：也叫cost默认0作用于两个相邻的as之间，进来的流量，越低越优先

公认团体属性：
- no_export                      不出as
- no_advertise                  不出route
- no_export_subconfed    不出联盟
bgp路径选择：
- 1、吓一跳b不可达，忽略
- 2、preferred-value数值高越好
- 3、local-prefence数值越高越好
- 。。。。

---
## 正则表达式
团体属性过滤列表（community-filter） 
- 基本只能按标准格式书写
- 高级可以使用正则表达式


### 2. 最核心三类元字符
- `.` 任意单个字符（除换行）
- `\d` 数字 0-9
- `\w` 字母、数字、下划线
- `\s` 空白（空格、制表、换行）
- `[]` 字符集，`[0-9a-z]` 匹配括号内任一字符

#### （2）量词（控制出现次数）
- `*` 0次或多次
- `+` 1次或多次
- `?` 0次或1次
- `{n}` 恰好n次；`{n,m}` n~m次

#### （3）位置与分组
- `^` 行开头，`$` 行结尾
- `()` 分组，可捕获内容
- `|` 或，`a|b` 匹配a或b

### 3. 经典最简示例
1. 手机号简单校验：`^1[3-9]\d{9}$`
2. 邮箱简易规则：`^\w+@\w+\.\w+$`
3. 匹配纯数字：`^\d+$`

## as_path 过滤
---
## 路由反射
路由反射蔟
防环靠cluster id 

---
## BGP原理
### 1.bgp报文


### 2.bgp状态机



### 3.bgp路由
1）**路由注入**：
- bgp不会自己产生路由，需要路由注入
- network ip mask
- import-route ospf/static/isis....
2）**路由聚合**：
- aggergate ip mask detail-suppressed 明细压制
- 策略针对聚合
### 4.通告原则

- BGP通过network、import‑route、aggregate聚合方式生成BGP路由后，通过Update报文将BGP路由传递给对等体。
 **BGP通告遵循以下原则**： 
1. 只发布最优路由:              
	- **display bgp routing-table**
	- * : 代表有效
	- >: 代表最优
2. 从EBGP对等体获取的路由会发布给所有对等体
3. IBGP水平分割：从IBGP对等体获取的路由，不会发送给IBGP对等体。
4. BGP同步规则指的是：ibgp路由不会直接通告给对端的ebgp路由器，除非ibgp对端有igp路由，即ibgp与igp路由同步，以防止路由黑洞
#### 5.路径属性
**AS_PATH**:
- apply as_path 300 additive            左侧追加
- apply as_path 300 overwrite          已有的替换为此处的
- apply as_path none overwrite        清空
**origin**：
- igp              标记i                **优先级i>e>?**
- egp             标记e
- inconplete  标记？
**next_hop**:
- 向ebgp发或本地始发向ibgp，值该为本地建立接口地址
- 收到EBGP所发的路由后，将其发给对等IBGP路由器时，下一跳不变
- 收到的bgp路由与自身的ebgp对等体处于同一网段，那么将他发布给其他的对等体时下一跳不变
**local_preference**:
- 越大越优先
- 只传递给ibgp对等体
- dis bgp routing-table
- 可以在as边界使用import方向策略来修改
- bgp default local-perference 使用默认100
- 引入的路由的本地优先级为默认优先级
**community**：
- 团体属性 格式 区域号：自定义号
- internet：默认属性
- NO_advertise:不宣告路由
- NO_Export:不出自治系统
- NO_Export_subconfed：不出子系统
**med**：
- 指出进入本as的最佳路径
- 越小越优
- 由ebgp传递到as后在as内传递携带该值，再次传向Ebgp时缺省不携带该值
- 向ebgp传递时是否有med：
	- 始发路由会有
	- 次发不会（即不会跨路由传递）
	- 非路由策略在ibgp间不丢失不改变
- 引入ospf值为100 直连为0
- 使用default med修改 **自身引入或聚合** 的路由
**atomic_Aggregate**
- 公认必遵
- 用于在路由表中表示是否是聚合路由
**Aggregate**：
- 可选过渡
- 聚合设备的as号
**preferred-value**：
- 协议首选值华为特有
- 仅在本地有效（不进行任何传递），当bgp路由表中存在des相同的条目选此值大的（0-65535）
#### 6.路由反射
- 解决水平分割ibgp路由器无法完全学习问题
- 角色：
	- rr ：反射路由器
	- client：rr的客户端
- 因为有反射器破坏了水平分割所以需要防环
	- 由第一个rr为学习到的路由加上发送端的originator id
- 路由反射簇
	- 每个as允许有多个蔟，每个簇的id缺省时为rr的route id
	- 当rr收到含有自生蔟的路由时认为存在环路则忽略该路由g'ne