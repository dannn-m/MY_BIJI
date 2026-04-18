# 一、工作方式
DHCPv4使用c/s模式，当客户端与服务器通信时，服务器会将IPv4地址分配或出租给客户端，直到租期满。客户端必须定期联系DHCP服务器延续租期。租期满后DHCP服务器会将地址返回地址池
# 二、获取租约的步骤
1. DHCP发现（DHCPDISCOVER)
2. DHCP提供（DHCPOFFER）
3. DHCP请求（DHCPREQUEST）
4. DHCP确认（SHCPACK)
## 1、发现
客户端使用包含自己MAC地址的广播消息来启动整个过程，查找可用的DHCP服务器，DHCPDISCOVER消息的目的是在网络中查找DHCPv4服务器
## 2、提供
服务器收到消息后会保留一个可用的ipv4地址以租赁给客户端。服务器还会创建一个arp条目，该条目包含请求客户端的MAC地址和客户端的租用IPv4地址。DHCP服务器会把绑定DHCPOFFER消息发送到请求客户端
## 3、请求
客户端收到提供消息时，客户端就会发回DHCPREQUEST播消息，此消息用于发起租用和租约更新。用于发起租用时，将SHCOERQUEST用作以提供参数所选定服务器的绑定接受通知，并隐式拒绝其他可能已为客户端提供绑定的服务器
## 4、确认
收到请求消息后，服务器会使用ICMP ping测验这个地址有无设备使用，他也为客户端租用该地址创建出一个新的ARP条目，然后使用DHCPACK消息进行应答。
除消息类型字段不同外DHCPACK与DHCPOFFER一样，客户端接受DHCPACK消息后，他会记录下配置信息，并且对分配给他的地址进行ARP查找。如果没有ARP应答，客户端就知道ipv4地址是有效的，并开始像使用自己的地址一样该地址。
![[Pasted image 20260417221508.png]]
# 三、续订租约的步骤
## 1、请求
客户端会把DHCPREQUEST消息发送到最初提供地址的那台DHCPv4服务器，如果指定时间没有收到DHCPACK客户端会广播宁一个DHCPREQUEST这样，另一个DHCP服务器便可以延续租约
## 2、确认
服务器返回DHCP包来确认
# 4、配置
## 1、配置思科DHCPv4服务器
1. 排除IPv4地址
2. 定义一个DHCPv4地址池的名称
3. 配置DHCPv4地址池
### i、排除IPv4地址
- 除非配置了排除特定地址，否则**路由器将充当DHCPv4服务器**
- 地址池中的某些IPv4地址要分配给静态地址分配的网络设备。所以需要排除这些地址
```
Router(config)# ip dhcp excluded-address 
low-address [high-address]
```
可以指定低位地址（low-address）和高位地址（high-address）来排除某一个地址或某一个范围地址
排除的地址应包括分配给路由器、服务器、打印机和其他一斤手动配置给设备的地址。可以多次输入此命令
### ii、定义地址池名称
命名且让路由器进让DHCPv4配置模式
```
Router(config)# ip dhcp pool _pool-name_  
Router(dhcp-config)#
```
### iii、配置地址池

| 任务                   | IOS 命令                                             |
| -------------------- | -------------------------------------------------- |
| 定义地址池。               | `network network-number [mask/ prefix-length]`     |
| 定义默认路由器或网关。          | `default-router address [ address2…address8]`      |
| 定义 DNS 服务器。          | `dns-server address [ address2…address8]`          |
| 定义域名。                | `domain-name domain`                               |
| 定义 DHCP 租期的持续时间。     | `lease {days [hours [ minutes]]infinite}`          |
| 定义 NetBIOS WINS 服务器。 | `netbios-name-server address [ address2…address8]` |
### iiii、示例
![[Pasted image 20260418101533.png]]
```
R1(config)# ip dhcp excluded-address 192.168.10.1 192.168.10.9

R1(config)# ip dhcp excluded-address 192.168.10.254

R1(config)# ip dhcp pool LAN-POOL-1

R1(dhcp-config)# network 192.168.10.0 255.255.255.0

R1(dhcp-config)# default-router 192.168.10.1

R1(dhcp-config)# dns-server 192.168.11.5

R1(dhcp-config)# domain-name example.com

R1(dhcp-config)# end

R1#
```
### v、验证命令
| 命令                                    | 描述                                         |
| ------------------------------------- | ------------------------------------------ |
| `show running-config \| section dhcp` | 显示路由器上配置的 DHCPv4 相关命令。                     |
| `show ip dhcp binding`                | 显示所有由 DHCPv4 服务提供的 IPv4 地址与 MAC 地址绑定关系的列表。 |
| `show ip dhcp server statistics`      | 显示关于已发送和接收的 DHCPv4 消息的数量信息。                |
