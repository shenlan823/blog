---
title: HCIA -- 11 -- RIP
date: 2023-09-02 22:15:31
tags:
  - HCIA
  - RIP
---
# 一、动态路由的概念

1. 网络中的路由器彼此之间相互通信，传递各自的路由信息，利用收到的路由信息来更新和维护自己路由表的过程，称之为动态路由。
2. 动态路由是基于某种动态路由协议实现的
   1. 单播网络环境中的6大动态路由协议：
    - RIP、OSPF、IGRP、EIGRP、IS-IS、BGPv4
   2. 组播网络环境中的6大动态路由协议：
    - PIM、IGMP、CGMP、MOSPF、DVMRP、CBT
3. 特点
   - 优点：自主性强，无需管理员手动干预，错误率低
   - 缺点：智能性过高导致可控性较低，学习路由条目时要消耗处理资源与带宽资源

# 二、动态路由协议

1. 向其它路由器传递路由信息
2. 接收其它路由器的路由信息
3. 根据收到的路由信息计算出到每个目的网络的最优路径，并由此生成路由表
4. 根据网络拓扑变化及时调整路由表，同时想其他路由器宣告拓扑改变的信息 

# 三、动态路由协议的分类

1. 按照使用的范围进行分类：
   - 内部网关路由协议【IGP】：RIP、OSPF、IGRP、EIGRP、IS-IS
   - 外部网关路由协议【EGP】：BGPv4
2. 按照算法进行分类：
   - 距离矢量路由协议：
     - 路由器每经过特定时间周期向邻居发送自己的路由表
     - 有距离(跳数)，有方向性(下一跳)；至于下一跳路由器是否能到达目标网络，本地路由器并不知晓【RIP】
   - 链路状态路由协议：
     - 每台路由器都自主学习整张网络拓扑结构环境，拥有完整的路由表项；之后再以自身根节点，计算出到大其它网络节点的最佳路径，由此生成路由表项【OSPF】
   - 混合型路由协议：
     - 同时具有距离矢量协议的特点，又具有链路状态协议的特点【IGRP、EIGRP】

# 四、RIP【路由信息协议】

## RIP的基本概念

1. TCP/ip参考模型下被开发出来的第一款动态路由协议
2. RIP属于标准的距离矢量动态路由协议
3. RIP【Routing Information Protocol】

## RIP的工作原理

![工作原理](../../images/HCIA/RIP/RIP工作原理.png)

1. 运行RIP的路由器平均每隔30s向其直连的邻居路由器发送整张路由表【若所有RIP路由器在同一时间同时发送路由更新的话，将会造成更新碰撞问题，因此在RIP中引入了一个小的随机抖动计时器（15%），因此每台RIP路由器的更新时间都是不同的（25s-35s）】
2. RIP以【跳数】作为其唯一的选路依据(哪近走哪)
3. RIP最多支持15跳（16个路由器），第16跳被标记为不可到达
4. 当网络中的所有路由器的路由表项完全一致后，该网络称之为达到了收敛状态，收敛的网络才能正常的转发数据

## RIP中的3个计时器

1. 周期更新计时器：30s
2. 路由老化计时器：180s（若在30s内没有从邻居路由器那里收到整张路由表的更新，则本地路由器将从对端那里学到的路由条目置为16跳，在等待150s）
3. 垃圾收集计时器：120s（在180s的路由老化计时器到时后，本地路由器在等待120s，若120s后依旧没有收到对端的路由表更新，则彻底删除之前学到的路由条目）

`注：更新计时器与老化计时器是同时开启的`

# 五、RIP的路由环路

- 环路产生的原因：路由器从一个接口接收到的路由更新，在发送更新时，又将该路由回传回去，从而容易引起环路的发生。
- 解决方案：
  1. 水平分割：
     - 保证路由器从一个接口接收到的路由信息，在更新时就不再回传回去，从而保障路由不存在环路问题。
  2. 触发更新：
     - 只能配置在串行链路上（Serial口）
  3. 路由毒化：
  4. 毒性逆转：

# 六、有类路由与无类路由

1. 有类路由：在传递路由更新时，不携带子网掩码
2. 无类路由：在传递路由更新时，携带子网掩码

# 七、RIP的版本

1. RIPv1：有类路由，不携带子网掩码；广播更新，广播地址：`255.255.255.255`
2. RIPv2：无类路由，携带子网掩码；组播更新，组播地址：`224.0.0.9`
3. RIPng：IPv6

# 八、实验配置

![RIP](../../images/HCIA/RIP/RIP.png)

**R1**

```bash
[R1]rip
[R1-rip-1]version 2
#默认使用RIPv1
#更改为RIPv2
[R1-rip-1]network 192.168.1.0
[R1-rip-1]network 10.1.1.0
Error: The network address is invalid, and the specified address must be major-n
et address without any subnets.
#华为的路由器不允许直接通告子网
#10.1.1.0是A类地址10.0.0.0的子网
#10.0.0.0是主网
[R1-rip-1]network 10.0.0.0
[R1-rip-1]undo summary  #不做汇总
#不将子网汇总成主网
```

```bash
[R1]dis ip routing-table 
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Tables: Public
         Destinations : 12       Routes : 12       

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

       ······
       20.1.1.0/24  RIP     100  1           D   10.1.1.2        GigabitEthernet
0/0/1
       ······
     172.16.1.0/24  RIP     100  2           D   10.1.1.2        GigabitEthernet
0/0/1
       ······
```

**R2**

```bash
[R2]rip
[R2-rip-1]version 2
[R2-rip-1]network 10.0.0.0
[R2-rip-1]network 20.0.0.0
[R2-rip-1]undo summary 
```

```bash
[R2]dis ip routing-table
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Tables: Public
         Destinations : 12       Routes : 12       

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

       ······
     172.16.1.0/24  RIP     100  1           D   20.1.1.2        GigabitEthernet
0/0/1
    192.168.1.0/24  RIP     100  1           D   10.1.1.1        GigabitEthernet
0/0/0
       ······
```

**R3**

```bash
[R3]rip	
[R3-rip-1]version 2
[R3-rip-1]network 20.0.0.0
[R3-rip-1]network 172.16.0.0	
[R3-rip-1]undo summary 
```

```bash
[R3]dis ip routing-table 
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Tables: Public
         Destinations : 12       Routes : 12       

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

       10.1.1.0/24  RIP     100  1           D   20.1.1.1        GigabitEthernet
0/0/0
       ······
0/0/1
    192.168.1.0/24  RIP     100  2           D   20.1.1.1        GigabitEthernet
0/0/0
       ······
```

**测试连通性**

```bash
PC>ping 172.16.1.10

Ping 172.16.1.10: 32 data bytes, Press Ctrl_C to break
From 172.16.1.10: bytes=32 seq=1 ttl=125 time=31 ms
From 172.16.1.10: bytes=32 seq=2 ttl=125 time=32 ms
From 172.16.1.10: bytes=32 seq=3 ttl=125 time=15 ms
From 172.16.1.10: bytes=32 seq=4 ttl=125 time=31 ms
From 172.16.1.10: bytes=32 seq=5 ttl=125 time=32 ms

--- 172.16.1.10 ping statistics ---
  5 packet(s) transmitted
  5 packet(s) received
  0.00% packet loss
  round-trip min/avg/max = 15/28/32 ms
```

## 关闭水平分割

- 水平分割功能开启：
  - R1：20.1.1.0/24  172.16.1.0/24
  - R3：10.1.1.0/24  192.168.1.0/24

```bash
[R2-GigabitEthernet0/0/0]undo rip split-horizon
[R2-GigabitEthernet0/0/0]int g0/0/1
[R2-GigabitEthernet0/0/1]undo rip split-horizon
#开启水平分割功能后只会传递没有的路由更新
#关闭水平分割功能后将会传递所有的路由更新
```