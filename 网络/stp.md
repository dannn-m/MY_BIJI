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
交换机每两秒发送BPDU帧