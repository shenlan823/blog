---
title: HCIA -- 12 -- OSPF
date: 2023-09-03 22:02:28
tags:
  - HCIA
  - OSPF
---

# 一、动态路由协议按使用范围进行分类

1. 内部网关路由协议【IGP】：RIPv1、RIPv2、OSPF、IGRP、EIGRP、IS-IS
2. 外部网关路由协议【EGP】：BGPv4

# 二、自治系统【AS】

1. 若一些路由器工作在同一个物理环境内，彼此之间都可以相互通讯，使用相同的路由选择协议，遵从相同的选路规则和依据，执行相同的路由策略，那么我们就称这些路由器工作在同一个自治系统【AS】内部。
2. 若一种路由协议完全工作在同一个 AS 内部，那么该协议就属于内部网关路由协议

# 三、OSPF的基础概念

1. OSPF没有路由器个数的限制，被称之为无限广大的路由协议。
2. OSPF为了防止单点故障引发的问题影响整个网络，因此其引入了区域【Area】的概念。
3. OSPF规定，每个区域中最多可以包含255台路由器，但是OSPF没有区域个数的限制。
4. 在OSPF中，区域0被称之为骨干区域，其他区域被称之为非骨干区域。
5. 非骨干区域必须能够通过一个ABR【Area Border Router｜区域边界路由器】与骨干区域直接相连，若没有ABR负责联通骨干区域与非骨干区域，则该非骨干区域无法与骨干区域通讯。
6. OSPF可以没有非骨干区域，所有路由器均工作在同一个区域0中（OSPF的单区域网络）
7. 若一台路由器的所有接口均工作在同一个区域中，则该路由器被称之为内部路由器；若该区域为区域0，则该路由器还可以被称为骨干路由器。
8. 若一台路由器的不同接口属于不同的区域，则该路由器被称之为区域边界路由器【ABR】，ABR的作用是用来在多个区域之间相互传递彼此的汇总路由信息。
9. OSPF区域的划分是在接口上的

# 四、Router-ID【路由器ID】

1. 每台路由器都会拥有一个独立且唯一的RID
2. 路由器选举RID的规则：
   1. 若路由器上配置了Loopback接口，则Loopback接口的地址就是该路由器的RID。若存在多个Loopback接口，则地址最大的成为RID。
   2. 若路由器上没有配置Loopback接口，则该路由器所有物理接口上，IP地址最大的将成为RID
   3. Loopbakc【本地回环接口】是一个虚拟接口，一般用途有2:
      1. 可以用来给路由器命名，在为路由器命名时，一般配置为32位的子网掩码。
      2. 用来模拟一个网段，或一个具体的主机，用来测试网络的联通性。

# 五、OSPF的3张表

1. 邻接关系表：保存的时自身直连邻居的路由器ID
2. 链路状态数据库表【LSDB｜Link State DataBase】：保存的是全网最完整的拓扑结构信息。
3. 路由表：保存的是到达每个网络节点的最佳路径

# 六、OSPF如何建立邻接关系

1. Down
2. Init
3. 2-way
4. Exstart
5. Exchange
6. Loading
7. Full

## 5种报文

1. Hello报文
2. DBD【DataBase｜数据库描述报文】
3. LSR【Link State Request｜链路状态请求报文】
4. LSU【Link State Update｜链路状态更新报文】
5. LSAck【Link State ACK｜链路状态确认报文】

# 七、OSPF建立邻接关系时需要满足的条件

1. 两台路由器的Area-ID要一致，相同区域可建立关系，不同区域不能建立关系。
2. OSPF可以支持配置密钥验证，若配置，则两端均需要配置，或不配置
3. 两台建立OSPF邻接关系的路由器的Hello时间与死亡时间必须一致【默认值：Hello - 10s；Dead - 40s；】

# 八、OSPF中的DR与BDR

1. 在一个广播型网络中，若所有路由器彼此之间都需要相互通讯，则各个路由器之间都需要建立邻接关系，建立的关系数量过于庞大，此方法不可取。
2. OSPF在整个网络中通过选择DR【指定路由器】与 BDR【备份制定路由器】的方式来缓解全互联的问题。
3. OSPF选择DR与BDR，先看优先级，优先级最高的成为DR，优先级第二高的成为BDR。
4. OSPF优先级的取值范围位：0 — 255，默认值为 1
5. 若所有OSPF路由器的优先级值相同，则根据RID选择，RID最大的成为DR，RID次大的成为BDR。
6. 优先级为0，表示不参与选举；优先级为255，表示直接胜出。
7. 其余的路由器均称为DROthers【其他路由器】，DROthers只与DR/BDR建立关系，DROthers之间处在2-way状态

# 九、OSPF的更新

1. OSPF使用与RIPv2一样的组播方式进行更新
2. OSPF占用2个组播地址：
   1.  DR/BDR：224.0.0.6
   2.  DROthers：224.0.0.5

# 十、OSPF的选路依据

1. OSPF根据带宽选路
   - Cost = 10^8^/BW
2. 计算时，使用10^8^除以换算成bit的带宽。
3. OSPF支持等价负载均衡，不支持非等价负载均衡。

# 十一、OSPF的优点

1. OSPF可适用于大型网络环境
2. OSPF属于触发更新，且有更新时只发送更新，没有更新时，10s发送一个Hello包
3. OSPF收敛速度快
4. OSPF支持区域的划分
5. OSPF属于无类路由，传递路由更新时会携带子网掩码
6. OSPF使用组播更新
7. OSPF确保选择的路径无环

# 实验配置

![OSPF](../../images/HCIA/OSPF/OSPF.png)

**R1**

```
[R1]ospf 1
[R1-ospf-1]area 0
[R1-ospf-1-area-0.0.0.0]network 192.168.1.0 0.0.0.255
[R1-ospf-1-area-0.0.0.0]network 10.1.1.0 0.0.0.255
```

**R2**

```bash
[R2]ospf 1
[R2-ospf-1]area 0
[R2-ospf-1-area-0.0.0.0]network 10.1.1.0 0.0.0.255
[R2-ospf-1-area-0.0.0.0]network 20.1.1.1 0.0.0.255
```

**R3**

```bash
[R3]ospf 1
[R3-ospf-1]area 0
[R3-ospf-1-area-0.0.0.0]network 20.1.1.2 0.0.0.255
[R3-ospf-1-area-0.0.0.0]network 172.16.1.10 0.0.0.255
```

**测试连通性**

```bash
PC>ping 172.16.1.10

Ping 172.16.1.10: 32 data bytes, Press Ctrl_C to break
From 172.16.1.10: bytes=32 seq=1 ttl=125 time=31 ms
From 172.16.1.10: bytes=32 seq=2 ttl=125 time=15 ms
From 172.16.1.10: bytes=32 seq=3 ttl=125 time=32 ms
From 172.16.1.10: bytes=32 seq=4 ttl=125 time=15 ms
From 172.16.1.10: bytes=32 seq=5 ttl=125 time=16 ms

--- 172.16.1.10 ping statistics ---
  5 packet(s) transmitted
  5 packet(s) received
  0.00% packet loss
  round-trip min/avg/max = 15/21/32 ms
```