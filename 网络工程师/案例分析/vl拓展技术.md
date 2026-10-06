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
- mux－vlan /