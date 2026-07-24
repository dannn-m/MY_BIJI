
---

## 一、创建 ACL（两种方式：编号 / 命名）

### 1. 编号 ACL

`acl [number] 编号`

- **基本ACL**：2000–2999
    
- **高级ACL**：3000–3999
    
- **二层ACL**：4000–4999
    

### 2. 命名 ACL

`acl name 名称 {basic | advanced | link}`

- basic：命名基本ACL
    
- advanced：命名高级ACL
    
- link：命名二层ACL
    

---

## 二、定义 Rule 规则（每类仅一条万能模板）

可选参数不写 = **匹配所有**

### 1. 基本 ACL（basic）仅匹配【源IP】

`rule [ rule-id ] {permit|deny} source {网段 反掩码 | host 单IP}`

### 2. 高级 ACL（advanced）匹配【源目IP/协议/端口】

`rule [ rule-id ] {permit|deny} {tcp|udp|icmp|ip} source 源 反掩码 destination 目的 反掩码 [eq 端口号]`

说明：`ip` = 所有IP协议；端口为可选

### 3. 二层 ACL（link）匹配【MAC地址】

`rule [ rule-id ] {permit|deny} source-mac MAC地址 MAC掩码`

---

## 三、接口应用（必须配置才生效）

`traffic-filter {inbound|outbound} acl {编号 | name ACL名称}`

- inbound：入接口过滤
    
- outbound：出接口过滤
    

---

## 四、查询/维护命令

```Plain
display acl all
display acl 编号
display acl name 名称
reset acl counter 编号/名称
```

---

## 五、极简核心口诀

- 基本ACL：只看源IP，放**靠近目的**
    
- 高级ACL：全字段匹配，放**靠近源**
    
- 不写匹配条件 = 匹配所有流量
    
- 不应用 traffic-filter = 策略完全不生效
    
- ACL 末尾默认隐含：全部拒绝