- 前缀列表匹配工具
- 天生用于匹配路由，不能在接口过滤数据
	- ip ip-prefix test index 10 permit 192.168.1.0 22 greater-equal 24 less-equal 26
	- 表示前 22 位必须精确匹配然后他的掩码范围在 24－26 之间
	- 缺省为不匹配所有
> []