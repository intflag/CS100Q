# 网络传输协议
- 参考资料：[刘超-趣谈网络协议【极客时间】](https://time.geekbang.org/column/intro/85)

## 网络协议综述

### 1）网络 5 层模型
![](http://images.intflag.com/protocol001.jpg)

### 2）工作流程
![](http://images.intflag.com/protocol002.jpg)

### 3）ip addr 命令
![](http://images.intflag.com/protocol004.jpg)

### 4）IP 地址
![](http://images.intflag.com/protocol003.jpg)
- 作用：提供远程定位功能，相当于「收货地址」
- 格式：IP 地址分为 4 个部分，每部分 8 位，共 32 位；
- 分类：5类（A类、B类、C类、D类、E类）
- 类型：公有地址、私有地址
- 无类别域间路由选择（CIDR）

```
IP/掩码位：16.158.165.91/22
子网掩码：255.255.252.0 (前22位表示网络号，后10位表示主机号)

网络号：16.158.164.0 (子网掩码和 IP 地址按位进行 AND 计算)
掩码：11111111.11111111.11111100.00000000  AND
地址：00010000.‬10011110.‭10100101‬.‭01011011‬
网络：00010000.10011110.10100100.00000000  = 16.158.164.0

第一个可用：16.158.164.1 (网络第一个)
二进制：00010000.10011110.10100100.00000001

最后一个可用：16.158.167.254 (网络倒数第二个)
二进制：00010000.10011110.10100100.11111110

广播地址：16.158.167.255 (网络最后一个)
二进制：00010000.10011110.10100100.11111111
```

### 5）MAC 地址
- 作用：区分全局唯一的功能 & 子网内定位功能
- 格式：6 字节，48 位，前 3 个字节（24 位）由 IEEE 分配，称为OUI（组织唯一标识符），后 3 个字节（24 位）由网卡制造商为其网卡分配一个唯一的编号
- MAC 地址的通信范围比较小，局限在一个子网里面

## 物理层

### 1）直通线与交叉线
- 直通线：不同设备之间的连接，网线两端的线序采用 568B 标准，统一都是：1、橙白 2、橙 3、绿白 4、蓝 5、蓝白 6、绿 7、棕白 8、棕
- 交叉线：相同设备之间的连接，网线一端采用 568B 标准，另一端采用 1-3、2-6 交叉接法，即 568A标准：1、绿白 2、绿 3、橙白 4、蓝 5、蓝白 6、橙 7、棕白 8、棕

### 2）HUB 集线器
- 可以组成一个最小的局域网 LAN (Local Area Network)
- 集线器没有大脑，它完全在物理层工作。它会将自己收到的每一个字节，都复制到其他端口上去

## 数据链路层
- 通信方式
    - 信道划分
    - 轮流协议
    - 随机接入协议，著名的以太网（Ethernet），用的就是这个方式
- 数据包格式
    - CRC：循环冗余检测，通过 XOR 异或的算法，来计算整个包是否在发送的过程中出现了错误

![](http://images.intflag.com/protocol005.jpg)

- Switch（交换机）
    - 在划分 vlan 的前提下可以实现多个广播域，每个接口都是一个单独的冲突域
    - 通过自我学习的方法可以构建出 CAM 表，并基于CAM进行转发数据
    - 支持生成树算法，可以构建出物理有环，逻辑无环的网络，网络冗余和数据传输效率都甩 Hub 好几条街，交换机是目前组网的基本设备之一

- ARP 协议：也就是已知 IP 地址，求 MAC 地址的协议

```
两台主机跨两个交换机 arp请求过程

- vm1 arping -b vm3
- sw1 学习到 vm1 是在自己左边的 #关注这个
- sw2 学习到 vm1 是在自己左边的
- vm3 回应 arp 单播
- sw2 根据转发表 给 SW1
- sw1 根据转发表 给 vm1
```
- RARP 协议：已知 MAC 地址求 IP 地址
- STP 协议：最小生成树协议
![](http://images.intflag.com/protocol006.jpg)

- VLAN 虚拟局域网
    - 在原来的二层的头上加一个 TAG，里面有一个 VLAN ID，一共 12 位，可以划分 4096 个 VLAN
    - 相同 VLAN 的包，才会互相转发，不同 VLAN 的包，是看不到的，可以解决广播问题

![](http://images.intflag.com/protocol007.jpg)

- Trunk 口：可以转发属于任何 VLAN 的口，交换机之间可以通过这种口相互连接

## 网络层
- ICMP(Internet Control Message Protoco)：互联网控制报文协议，ping 是基于 ICMP 协议工作的

![](http://images.intflag.com/protocol008.jpg)

- ICMP 报文类型
    - 查询报文：常用的 ping 就是查询报文，是一种主动请求，并且获得主动应答的 ICMP 协议，主动请求为 8，主动请求的应答为 0
    - 差错报文类型：终点不可达为 3，源抑制为 4，重定向为 5，超时为 11

- ping 流程
![](http://images.intflag.com/protocol009.jpg)

- Traceroute
    - 发 UDP 数据包，收 ICMP 数据包
    - 设置特殊的 TTL，来追踪去往目的地时沿途经过的路由器
    - 设置不分片，从而确定路径的 MTU

- 网关：通常是一个三层转发设备，所谓三层设备就是把 MAC 头和 IP 头都取下来，然后根据里面的内容，看看接下来把包往哪里转发的设备。

- 静态路由：静态配置的若干条路由规则（网络目标、网络掩码、网关、接口、跃点数）

- 转发网关（欧洲十国游）
    - MAC 改变，IP 不变
![](http://images.intflag.com/protocol100.jpg)

- NAT 网关 （玄奘西行）
    - MAC 改变，IP 也改变，由路由器做 NAT 转换

- 动态路由算法
    - 距离矢量路由（distance vector routing），基于 Bellman-Ford 算法
        - BGP（Border Gateway Protocol）外网路由协议，简称 BGP 协议
    - 链路状态路由（link state routing），基于 Dijkstra 算法
        - IGP（Interior Gateway Protocol）内部网关协议，简称 IGP 协议

## 传输层
### TCP 与 UDp
- TCP 是面向连接的，UDP 是面向无连接的，所谓面向连接是为了在客户端和服务端维护连接，建立了一定的数据结构来维护双方的状态；
- TCP 是面向字节流的，底层发送的是 IP 包，UDP 是基于数据报的；
- TCP 可以进行拥塞控制，并且是有状态的，记录发了没有，发到哪里，收到没有，UDP 是无状态服务；

### UDP 包格式
- 源端口号（16位），目的端口号（16位）；
- UDP 长度（16位）、UDP 校验和（16位）、数据；

### UDP 应用场景
- 内网质量好或者不需要建立连接，可以广播的场景，如 DHCP；
- 流媒体协议、物联网协议、4G 网络协议；

### TCP 包格式
- 源端口号（16位），目的端口号（16位）；
- 序号（32位），确认序号（32位）：解决乱序问题；
- 状态位：SYN 发起连接，ACK 回复，RST 重新连接，FIN 结束连接；
- 窗口大小（16位）、紧急指针（16位）：流量控制、拥塞控制；

### TCP 三次握手
请求 -> 应答 -> 应答之应答

![](http://images.intflag.com/protocol200.jpg)

### TCP 四次挥手

![](http://images.intflag.com/protocol201.jpg)

### TCP 状态机

![](http://images.intflag.com/protocol202.jpg)


### TCP 滑动窗口

![](http://images.intflag.com/protocol203.jpg)

### Socket
- 需要指定 IPv4 或 IPv6；
- 需要指定 TCP 或 UDP；

#### 基于 TCP 协议的 Socket 程序函数调用过程

![](http://images.intflag.com/protocol204.jpg)

#### 基于 UDP 协议的 Socket 程序函数调用过程

![](http://images.intflag.com/protocol205.jpg)

## 应用层

### HTTP
### HTTPS
### DNS
### HttpDNS
### CDN
### VPN
