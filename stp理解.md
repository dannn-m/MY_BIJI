# stp的工作流程
- 选举根桥
- 选举根端口
- 选举指定端口
- 其他端口禁用
# stp状态
- disable
- blocking
- listening
- learning
- forwording
首先当交换机开机以后他会进入listening状态，并互相发送以自己为根桥的bpdu报文

通过比较桥id（桥id由优先级+mac地址组成，也可理解为先比较优先级再比较mac）（越小越优）选出根桥

当下游的交换机发现自己收到的bpdu报文比自身的优时