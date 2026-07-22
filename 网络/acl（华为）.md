# 1. 创建 ACL
## 编号 ACL
~~~
acl number 编号
~~~
basic（基本 ACL）：2000～2999
advanced（高级 ACL）：3000～3999
link（二层 ACL）：4000～4999
命名 ACL
shell
acl name 名称 {basic | advanced | link}
basic：命名基本 ACL
advanced：命名高级 ACL
link：命名二层 ACL
2. 定义规则 rule
通用格式：rule {permit|deny} [匹配条件]
缺省规则：
不填写匹配条件 = 匹配所有流量
ACL 末尾隐含 rule deny，未匹配流量全部拒绝
basic 基本 ACL（仅源 IP）
shell
rule {permit|deny} source {网段 反掩码 | host 单IP}
advanced 高级 ACL（源目 IP、协议、端口）
shell
rule {permit|deny} {tcp|udp|icmp|ip} source 源网段 反掩码 destination 目的网段 反掩码 [eq 端口号]
ip 代表匹配全部 IP 协议
link 二层 ACL（MAC 地址过滤）
shell
rule {permit|deny} source-mac MAC地址 MAC掩码
3. 接口应用（必须配置才生效）
shell
traffic-filter {inbound|outbound} acl {编号 | name ACL名称}
inbound：入接口方向过滤
outbound：出接口方向过滤
辅助查询命令
shell
display acl all
display acl {编号 | name ACL名称}
reset acl counter {编号 | name ACL名称}
核心速记
basic：仅源 IP，部署靠近目的端
advanced：多条件精准匹配，部署靠近源端
link：二层 MAC 过滤
仅创建 ACL，不执行 traffic-filter，策略不生效