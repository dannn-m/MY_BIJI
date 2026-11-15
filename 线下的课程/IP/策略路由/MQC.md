- MQC(modular qos command－line interface，模块化 Qos 命令行)：指通过将某类具有共同特征的数据流划分为一类，并为同一类数据流量提供  相同服务
- 三要素：流分类、流行为和流策略
	- 流分类用于匹配感兴趣的流
	- ![[Screenshot_2026-11-15-12-43-46-23_769977972775e0c6b41aa3dfaf766445.jpg]]
	- 流行为支持重定向报文，因此使用 MQC 可以实现 IP 单播策略路由
		- 可执行的动作：过滤报文、流浪统计、为报文添加 VLAN tag 重标记报文优先级、对报文执行策略路由.....
	- 流策略
		- 有出方向与入方向
		- 在接口入方向应用策略
		- 对属于该 vlan 并匹配流分类中规则的入方向实施策略控制
		- 在全局或板卡上应用流策略
# 配置
1.创建流分类：
`trafic classifiler name 【operater 「and｜or」】`
缺省情况下，刘分类中各规则之间的关系为‘or’
2.创建流行为：
`trafic behavior name`
3.创建流策略，并 banding 流分类与流行为
`trafic policy name`
`classifier name behavior name`
