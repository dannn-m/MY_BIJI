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
- 不是根端口与指定端口的端口将会被