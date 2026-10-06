# super vlan/Aggregate－vlan
多个 vl 共用一个 ip 地址段相当于两个 vl 聚合过后共用一个网关
网关需要配置：
- vl 10（这是 super vl）
- aggregate－vlan
- access－vlan
- int vl 10
- arp－proxy inter－sub－vlan－proxy en
# mux－vlan
- vlan 100
- mux－vlan /启用 mux vl 并 100 为主 vl
- subordinate sparate 200 //将 200 设置为隔离 vl（vl 内主机互相隔离）
- subordinate sparate 300 //将 300 设置为组 vl（vl 内主机互通，不同组间隔离）
- 需要在
> [!note]
> 隔离 vl 只能有一个，这些子 vl 都可以与主 vl 通


