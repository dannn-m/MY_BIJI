# 一、工作方式
DHCPv4使用c/s模式，当客户端与服务器通信时，服务器会将IPv4地址分配或出租给客户端，直到租期满。客户端必须定期联系DHCP服务器延续租期。租期满后DHCP服务器会将地址返回地址池
# 二、获取租约的步骤
1. DHCP发现（DHCPDISCOVER)
2. DHCP提供（DHCPOFFER）
3. DHCP请求（DHCPREQUEST）
4. DHCP确认（SHCPACK)
