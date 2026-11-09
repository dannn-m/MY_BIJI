- 前缀列表匹配工具
- 天生用于匹配路由，不能在接口过滤数据
	- ip ip-prefix test index 10 permit 192.168.1.0 22 greater-equal 24 less-equal 26
	- 表示前 22 位必须精确匹配然后他的掩码范围在 24－26 之间
	- 若不写 greater－equal 那么 ip 后面的掩码就是范围的下限
> [!note] 是用路由在定义规则范围内查找，如果找到就停止匹配下一条规则，若所有认为定义的规则都不匹配，缺省匹配为 deny（因为缺省有 deny any 的规则）