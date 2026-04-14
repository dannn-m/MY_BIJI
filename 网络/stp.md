**前景**：以太网LAN要求拓扑是无环的，环路会导致以太网帧在环路中不断传播
# 一、STP 工作方式
## 1.创建无环拓扑步骤
STP会使用STA(生成树算法)通过四个步骤创造出无环的拓扑
1. 选举根桥
2. 选举根端口
3. 选举指定端口
4. 选举替代（阻塞）端口
在其工作中交换机会使用网桥协议数据单元（BPUD)来分享自己和自己链接的一切信息，其用于选举根桥，根端口，指定端口与替代端口，每个BPUD都包含一个BID用于标识发送这个交换机本身，BID中包含了优先级值、发送方的MAC地址、可选的拓展系统id。最低的BID值取决于这三个字段的组合
<div style="width: 90%; margin: 20px auto; font-family: 微软雅黑, sans-serif;"> <!-- 标题 --> <div style="text-align: left; margin-bottom: 15px; font-size: 20px; font-weight: bold;">带扩展系统ID的网桥ID (BID)</div> <!-- 分段色块容器（严格按4:12:48比例） --> <div style="display: flex; border: 2px solid #ccc; border-radius: 4px; overflow: hidden;"> <div style="flex: 4; background: #008080; color: #000; text-align: center; padding: 40px 0; font-size: 20px; font-weight: bold;">网桥优先级</div> <div style="flex: 12; background: #008080; color: #000; text-align: center; padding: 40px 0; font-size: 20px; font-weight: bold; border-left: 2px solid #fff;">扩展系统ID</div> <div style="flex: 48; background: #008080; color: #000; text-align: center; padding: 40px 0; font-size: 20px; font-weight: bold; border-left: 2px solid #fff;">MAC地址</div> </div> <!-- 箭头+位数标注（严格对应块宽度） --> <div style="display: flex; text-align: center; margin-top: 10px;"> <div style="flex: 4;"> <div style="border-bottom: 2px solid #ff6600; position: relative; height: 20px;"> <span style="position: absolute; left: 50%; transform: translateX(-50%); top: 10px; font-size: 18px;">4位</span> <span style="position: absolute; left: 0; top: -5px; font-size: 24px; color: #ff6600;">←</span> <span style="position: absolute; right: 0; top: -5px; font-size: 24px; color: #ff6600;">→</span> </div> </div> <div style="flex: 12;"> <div style="border-bottom: 2px solid #ff6600; position: relative; height: 20px;"> <span style="position: absolute; left: 50%; transform: translateX(-50%); top: 10px; font-size: 18px;">12位</span> <span style="position: absolute; left: 0; top: -5px; font-size: 24px; color: #ff6600;">←</span> <span style="position: absolute; right: 0; top: -5px; font-size: 24px; color: #ff6600;">→</span> </div> </div> <div style="flex: 48;"> <div style="border-bottom: 2px solid #ff6600; position: relative; height: 20px;"> <span style="position: absolute; left: 50%; transform: translateX(-50%); top: 10px; font-size: 18px;">48位</span> <span style="position: absolute; left: 0; top: -5px; font-size: 24px; color: #ff6600;">←</span> <span style="position: absolute; right: 0; top: -5px; font-size: 24px; color: #ff6600;">→</span> </div> </div> </div> <!-- 底部说明 --> <div style="margin-top: 30px; font-size: 18px;">这个 BID 中包含了网桥优先级、扩展系统 ID 和交换机的 MAC 地址，总长度为64位（8字节）。</div> </div>
- **网桥优先级**：思科默认32768。范围0到61440（增量为4096），数值越低优先级越高
- **拓展系统id**：是添加到BID中的十进制值，可以标识这个BPDU帧所属的VLAN
- **MAC**：当两台交换机配置相同的网桥优先级和相同的拓展系统id时，mac地址最低的有较小的BID
## 2.选举根桥
- 交换机每两秒发送BPDU帧，包含发送方的BID和根桥的BID，后者称根id
- 一开始所有交换机都会宣布自己就是根网桥，同时把自己的BID设置为根id，最终这些交换机通过学习选举最低根id的交换机为根网桥
![[Pasted image 20260413151500.png]]

## 2.确认根路径开销
- 选举根网桥后，STA会开始确定广播域中所有目的地到达根网桥的最佳路径。
- 把一台交换机到根网桥的路径沿途加在一起，得到的路径信息叫内部根路径开销
**注意**:BPDU包含了根路径开销。这是从发送该BPDU的交换机到达根桥的路径开销。当交换机收到BPDU时，他会添加网段入站的端口开销以确定其[[内部根路径开销]]

默认情况下，端口开销由端口工作速率决定，思科交换机默认使用IEEE 802.1D为STP和RSTP定义值，也称短路径开销。不过IEEE标准建议在使用10gbps及更高速率链路时，使用IEEE-802.1W中的值（也称长路径开销）
**表为ieee建议默认端口开销**

|链路速度|STP Cost: IEEE 802.1D-1998|RSTP Cost: IEEE 802.1w-2004|
|:--|:--|:--|
|10 Gbps|2|2,000|
|1 Gbps|4|20,000|
|100 Mbps|19|200,000|
|10 Mbps|100|2,000,000|
## 3.选择根端口
- 每个交换机都会选择一个根端口，它指从总开销方面看最接近根桥的端口，这个总开销也叫内部根路径开销
- 开销最短的会成为首选路径，其他冗余路径会被阻塞
## 4.选择指定端口
- 指定端口是一个网段上（两个交换机之间）的端口，它拥有去往根桥的[[内部根路径开销]]，也就是它具有接收发往根桥流量的最佳
- 不是根端口与指定端口的端口将会成为替代端口会或阻塞端口
![[Pasted image 20260413155334.png]]

## 5.从登开销路径选根端口 
1. 最低发送方的BID
2. 最低发送方端口优先级
3. 最低发送方端口ID
## 6.STP计时器和端口状态
STP 的收敛需要三个计时器，包括：
- Hello 计时器 - hello 时间是 BPDU 之间时间的间隔。这个值默认为 2 秒，不过可以修改为 1 到 10 秒之间的值。
- 转发延迟计时器 - 转发延迟是在侦听和学习状态中消耗的时间。这个值默认为 15 秒，不过可以修改为 4 到 30 秒之间的值。
- 最大老化时间计时器 - 最大老化时间是交换机在尝试修改 STP 拓扑之前，等待的最大时间长度。这个值默认为 20 秒，不过可以修改为 6 到 40 秒之间的值。
注意: 默认时间可以在根桥上更改，用来表示这个 STP 域的计时器值。
STP 用于为整个广播域确定逻辑无环路径。互连的交换机通过交换 BPDU 帧来获知信息，生成树即是根据这些信息而确定的。如果交换机端口直接从阻塞状态转换到转发状态，而转换过程中没有关于完整拓扑的信息，那么端口会临时形成数据环路。出于这种原因，STP 有五种端口状态，其中四种是正常运行的端口状态，如图所示。禁用状态可以视为是非工作状态。
![[Pasted image 20260413161153.png]]

|端口状态|描述|
|:--|:--|
|**阻塞**|这种端口是替代端口，不参与帧转发。端口接收 BPDU 帧以确定根桥的位置和根 ID。BPDU 帧还会用于判断每个交换机端口 应该在最终的活动 STP 拓扑中承担什么端口角色。由于 最大老化时间计时器为 20 秒，没有从邻居交换机那里 如期接收到 BPDU 的交换机端口就会进入阻塞状态。|
|**侦听**|在阻塞状态之后，端口就会进入侦听状态。其中 端口接收 BPDU 以确定到根的路径。交换机端口 也会发送自己的 BPDU 帧，并通知相邻的交换机 这个交换机端口准备参与到活动拓扑当中。|
|**学习**|在侦听状态后，交换机端口会过渡到 学习状态。在学习状态中，交换机端口会接收并处理 BPDU，同时准备参与帧的转发。端口也会开始 填充 MAC 地址表。但是，在学习状态下，用户 帧不会被转发给目的地。|
|**转发**|在转发状态下，交换机端口可以视为参与了活动 拓扑。交换机端口会转发用户流量，同时发送和接收 BPDU 帧。|
|**已禁用**|处于禁用状态的交换机端口不会参与生成树，并且也不会转发帧。当交换机端口被管理禁用时，这个端口就会设置为禁用状态。|

| 端口状态   | BPDU    | MAC 地址表 | 转发数据帧 |
| :----- | :------ | :------ | :---- |
| **阻塞** | 仅接收     | 无更新     | 否     |
| **监听** | 接收并发送   | 无更新     | 否     |
| **学习** | 接收并发送   | 更新表     | 否     |
| **转发** | 接收并发送   | 更新表     | 是     |
| **禁用** | 不发送且不接收 | 无更新     | 否     |

# 二、stp的演变
## 1.不同stp的变体
|协议|收敛速度|VLAN 支持|厂商属性|核心定位|
|---|---|---|---|---|
|STP|慢（30-50s）|所有 VLAN 共用 1 棵树|通用标准|初代基础版，淘汰|
|PVST+|慢（30-50s）|每个 VLAN1 棵树|思科私有|按 VLAN 分树，但慢|
|RSTP|快（1s 内）|所有 VLAN 共用 1 棵树|通用标准|解决 STP 慢的问题|
|802.1D-2004|快（1s 内）|所有 VLAN 共用 1 棵树|通用标准|STP+RSTP 的合集标准|
|Rapid PVST+|快（1s 内）|每个 VLAN1 棵树|思科私有|思科设备的 “快 + 按 VLAN 分树” 方案|
|MSTP|快（1s 内）|多 VLAN 分组共用 1 棵树|通用标准|现在的主流，兼顾速度、负载、兼容性|
|MST|快（1s 内）|多 VLAN 分组共用 1 棵树|思科私有实现|思科版 MSTP|
## 2、RSTP的概念
### RSTP的端口状态
\#与是stp类似
![[Pasted image 20260414151210.png]]
### PortFast和BPDU防护
当交换机端口或交换机上电时，交换机端口会经历侦听和学习状态，mei'ci