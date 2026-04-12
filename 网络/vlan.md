---
tags:
  - 网络
date:
---
## 关于语音和数据vlan
![[Pasted image 20260408091239.png]]
- 一个接口只能划分给一个vlan但是端口却再可以与语音关联
使用 **switchport voice vlan** _vlan-id_ 接口配置命令把一个语音 VLAN 分配给端口。

支持语音流量的 LAN 一般都会启用服务质量 （QoS）。语音流量在进入网络时就必须标记为可信任。使用 **mls qos trust \[cos | device cisco-phone | dscp | ip-precedence]** 接口配置命令设置接口的可信状态，并指示数据包中的哪些字段会用来对流量进行分类。

示例中的配置会创建出两个 VLAN （即 VLAN 20 和 VLAN 150），然后将 S3 的 F0/18 接口作为交换机端口划分到 VLAN 20 中。还将语音流量分配给 VLAN 150，并根据 IP 电话分配的服务类别 (CoS) 启用 QoS 分类。

```
S3(config)# **vlan 20**

S3(config-vlan)# **name student**

S3(config-vlan)# **vlan 150**

S3(config-vlan)# **name VOICE**

S3(config-vlan)# **exit**

S3(config)# **interface fa0/18**

S3(config-if)# **switchport mode access**

S3(config-if)# **switchport access vlan 20**

S3(config-if)# **mls qos trust cos**

S3(config-if)# **switchport voice vlan 150**

S3(config-if)# **end**
```

# 一、传统VLAN间路由
- 通关将交换机连接到路由器上，路由器的每个接口连接一个vlan，这个接口充当这个vlan的本地网关
![[Pasted image 20260407174842.png]]
- 缺点：路由器接口有限无法承载太多vlan，以及拓展性不佳
# 二、单臂路由器vlan间路由

## 1.概念
- 路由器只需连接在vlan间的中继上，不同vlan间的通信都由路由器来路由
## 2.原理
- 路由器的物理以太网接口配置了多个虚拟子接口
- 每个子接口独立配置ip并划分vlan这些子接口需要按照划分的vlan来配置不同的子网地址
- 当流量进入路由器时就会被转发给vlan子接口并打上对应的标签发送回去
## 3.缺点
- 容错低当单臂线路断了后各vlan间不能通信
- 多个vlan使用一条线路易堵塞
## 3.配置
![[Pasted image 20260407183944.png]]
###  a.s1创建vlan
- 首先，创建并命名 VLAN。只有在退出 [[VLAN 子配置模式]]之后， VLAN 才会创建出来。
```
S1(config)# vlan 10
S1(config-vlan)# name LAN10
S1(config-vlan)# exit
S1(config)# vlan 20
S1(config-vlan)# name LAN20
S1(config-vlan)# exit
S1(config)# vlan 99
S1(config-vlan)# name Management
S1(config-vlan)# exit
S1(config)#
```
### b.s1创建管理接口

接下来，在 VLAN 99 上创建管理接口，并且指定 R1 作为默认网关。

```
S1(config)# interface vlan 99
S1(config-if)# ip add 192.168.99.2 255.255.255.0
S1(config-if)# no shut
S1(config-if)# exit
S1(config)# ip default-gateway 192.168.99.1
S1(config)#
```
### c. s1配置接入端口。

接下来，把连接到 PC1 的 Fa0/6 端口配置为 VLAN 10 中的接入端口。假定 PC1 上已经配置了正确的 IP 地址和默认网关。

```
S1(config)# interface fa0/6
S1(config-if)# switchport mode access
S1(config-if)# switchport access vlan 10
S1(config-if)# no shut
S1(config-if)# exit
S1(config)#
```
### d. s1配置中继端口。

最后，把连接到 S2 的端口 Fa0/1 和连接到 R1 的 Fa05 端口配置为中继端口。

```
S1(config)# interface fa0/1
S1(config-if)# switchport mode trunk
S1(config-if)# no shut
S1(config-if)# exit
S1(config)# interface fa0/5
S1(config-if)# switchport mode trunk
S1(config-if)# no shut
S1(config-if)# end
*Mar 1 00:23:43 .093：%LINEPROTO-5-UPDOWN： Line protocol on Interface FastEthernet0/1， changed state to up
*Mar 1 00:23:44 .511：% LINEPROTO-5-UPDOWN： Line protocol on Interface FastEthernet 0/5， changed state to up
```
### e.s2的vlan与中继配置
与s1配置差不多
~~~
S2(config)# **vlan 10**
S2(config-vlan)# **name LAN10**
S2(config-vlan)# **exit**
S2(config)# **vlan 20**
S2(config-vlan)# **name LAN20**
S2(config-vlan)# **exit**
S2(config)# **vlan 99**
S2(config-vlan)# **name Management**
S2(config-vlan)# **exit**
S2(config)#
S2(config)# **interface vlan 99**
S2(config-if)# **ip add 192.168.99.3 255.255.255.0**
S2(config-if)# **no shut**
S2(config-if)# **exit**
S2(config)# **ip default-gateway 192.168.99.1**
S2(config)# **interface fa0/18**
S2(config-if)# **switchport mode access**
S2(config-if)# **switchport access vlan 20**
S2(config-if)# **no shut**
S2(config-if)# **exit**
S2(config)# **interface fa0/1**
S2(config-if)# **switchport mode trunk**
S2(config-if)# **no shut**
S2(config-if)# **exit**
S2(config-if)# **end**
*Mar  1 00:23:52.137: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to up
~~~
### f.R1的子接口配置
- 子接口使用全局命令模式的**interface** interface_id subinterface_id创建
- 然后使用下面两条命令对每个子接口进行配置：

	- **encapsulation dot1q** _vlan_id_ **\[native\]** - 此命令将子接口配置为响应来自指定 _vlan-id_ 的 802.1Q 封装的流量. 仅附加了 **native** 关键字选项，以将本地 VLAN 设置为 VLAN 1 以外的其他端口。
	- **ip address** _ip-address subnet-mask_ - 这条命令的作用是给子接口配置 IPv4 地址。这个地址一般会充当对应 VLAN 的默认网关。
192.168.10.10/24G0/0/1F0/5192.168.99.2/24F0/1F0/1192.168.99.3/24F0/6F0/18192.168.20.10/24S1S2R1PC1PC2


```
R1(config)# interface G0/0/1.10
R1(config-subif)# description Default Gateway for VLAN 10   #此句为注释语句语法为discription 描述内容
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip add 192.168.10.1 255.255.255.0R1(config-subif)# exit
R1(config)#
R1(config)# interface G0/0/1.20
R1(config-subif)# description Default Gateway for VLAN 20
R1(config-subif)# encapsulation dot1Q 20
R1(config-subif)# ip add 192.168.20.1 255.255.255.0
R1(config-subif)# exit
R1(config)#
R1(config)# interface G0/0/1.99
R1(config-subif)# description Default Gateway for VLAN 99
R1(config-subif)# encapsulation dot1Q 99
R1(config-subif)# ip add 192.168.99.1 255.255.255.0
R1(configsubif)# exit
R1(config)#
R1(config)# interface G0/0/1
R1(config-if)# description Trunk link to S1
R1(config-if)# no shut
R1(config-if)# end
R1#*Sep 15 19:08:47.015: %LINK-3-UPDOWN: Interface GigabitEthernet0/0/1, changed state to down*Sep 15 19:08:50.071: %LINK-3-UPDOWN: Interface GigabitEthernet0/0/1, changed state to up*Sep 15 19:08:51.071: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up
R1#
```
### g.故障排除
- **show ip route**
- **show ip interface brief**
- **show interfaces**
- show interfaces trunk
# 三、使用三层交换机实现vlan间路由
## 1.概念
- 三层交换机可工作在网络层实现一定路由功能
## 2.原理
执行 VLAN 间路由的方法是使用第 3 层交换机和交换虚拟接口 (SVI)。如图所示，SVI 是配置在多层交换机上的一种虚拟接口。
![[Pasted image 20260407190418.png]]
**注意:** 第 3 层交换机也称为多层交换机，因为它工作在第 2 层和第 3 层。



VLAN 间 SVI 的创建方式与管理 VLAN 接口的配置方式相同。这个 SVI 是给交换机上的一个 VLAN 创建的。虽然是虚拟接口，但是 SVI 可以为 VLAN 执行与路由器接口相同的功能。具体来说，它可以为往返于这个 VLAN 中所有交换机端口的数据包提供第 3 层处理功能。

## 3.优点

- 因为所有操作都是通过硬件进行交换和路由的，所以这种方式比单臂路由器要快很多。
- 无需在交换机和执行路由转发的路由器之间连接外部链路。
- 不受单一链路的限制，因为第 2 层以太通道可以充当交换机之间的中继链路，来增加带宽。
- 延迟更低，这是因为数据无需离开交换机就可以路由到另一个网络中。
- 比起路由器，它们在园区局域网中的使用更加普遍。
## 4.配置
### **1. 创建 VLAN。**

首先，创建两个 VLAN，如输出信息所示。

```
D1(config)# vlan 10
D1(config-vlan)# name LAN10
D1(config-vlan)# vlan 20
D1(config-vlan)# name LAN20
D1(config-vlan)# exit
D1(config)#
```
### **2. 创建 SVI VLAN 接口。**

为 VLAN 10 和 VLAN 20 配置 SVI。配置的 IP 地址会充当对应 VLAN 中主机的默认网关。可以看到消息显示，两个 SVI 上面的线路协议都已经启动 （up）。

```
D1(config)# interface vlan 10
D1(config-if)# description Default Gateway SVI for 192.168.10.0/24
D1(config-if)# ip add 192.168.10.1 255.255.255.0
D1(config-if)# no shut
D1(config-if)# exit
D1(config)#
D1(config)# int vlan 20
D1(config-if)# description Default Gateway SVI for 192.168.20.0/24
D1(config-if)# ip add 192.168.20.1 255.255.255.0
D1(config-if)# no shut
D1(config-if)# exit
D1(config)#
*Sep 17 13:52:16.053: %LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan10, changed state to up
*Sep 17 13:52:16.160: %LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan20, changed state to up
```
### **3. 配置接入端口。**

接下来，配置连接主机的接入端口，并把这些端口划分到各自的 VLAN 中。

```
D1(config)# interface GigabitEthernet1/0/6
D1(config-if)# description Access port to PC1
D1(config-if)# switchport mode access
D1(config-if)# switchport access vlan 10
D1(config-if)# exit
D1(config)#
D1(config)# interface GigabitEthernet1/0/18
D1(config-if)# description Access port to PC2
D1(config-if)# switchport mode access
D1(config-if)# switchport access vlan 20
D1(config-if)# exit
```
### **4. 启用 IP 路由。**

最后，使用 **ip routing** 全局配置命令启用 IPv4 路由，让流量可以在 VLAN 10 和 20 之间进行交互。这条命令必须配置，这样才能在第 3 层交换机上针对 IPv4 协议启用 VLAN 间路由。

```
D1(config)# ip routing
D1(config)#
```


# 4.故障排除
## 常见问题及验证
| 问题类型       | 如何修复                                                             | 如何验证                                                                                  |
| :--------- | :--------------------------------------------------------------- | :------------------------------------------------------------------------------------ |
| 缺失 VLAN    | ・如果 VLAN 不存在则创建（或重新创建）VLAN。<br><br>・确保将主机端口划分到了正确的 VLAN。         | `show vlan [brief]`<br><br>`show interfaces switchport`<br><br>`ping`                 |
| 交换机中继端口的问题 | ・确保中继配置正确。<br><br>・确保端口是中继端口并且已经启用。                              | `show interfaces trunk`<br><br>`show running-config`                                  |
| 交换机接入端口的问题 | ・给接入端口分配正确的 VLAN。<br><br>・确保端口是一个接入端口并且已经启用。<br><br>・主机配置了错误的子网。 | `show interfaces switchport`<br><br>`show running-config interface`<br><br>`ipconfig` |
| 路由器配置问题    | ・纠正路由器子接口的 IPv4 地址配置错误。<br><br>・给路由器子接口分配正确的 VLAN ID。            | `show ip interface brief`<br><br>`show interfaces`                                    |