---
title: HCIA -- 10 -- 静态路由
date: 2023-09-02 19:11:54
tags:
  - HCIA
  - 静态路由
---

# 一、路由的概念

- 跨越从源主机到目标主剧的一个互联网络来转发数据包的过程
- 在转发过程中，负责路由数据包的机器，称之为路由器

# 二、路由器的工作原理

1. 路由器与交换机相比，在转发数据时，唯一的相同点在于，都会查表转发【路由器：路由表；交换机：MAC 地址吧】
2. 路由器与交换机在转发过程中的区别：
   1. 交换机会学习源 MAC 地址，通过不断的学习源 MAC 地址来完整自身的 MAC地址表象；路由器不学习源 IP 地址，只关心目标网络。
   2. 交换机在无法找到目标网络时会进行广播转发操作，而路由器若没有找到路由表，则执行丢弃操作，向信源返回不可达信息。

## 路由表

- 在路由器中维护的路由条目，路由器根据路由表做路径选择。

## 直连路由

- 当在路由器上配置了借口的IP地址，并且接口状态为up的时候，路由表中就出现直连路由表项。

# 三、路由表的形成

1. 路由器不学习源IP地址
2. 路由器在配置好自身的接口IP地址后，首先会学习自身接口IP地址所属的网段，形成直连路由。
3. 对于非直连的路由，其默认是不会出现在我的本地路由表中的，若想知道非直连的路由，则需要：
   1. 静态路由：
   2. 动态路由：RIP、OSPF、IGRP、EIGRP、IS-IS、BGPv4

>注：在路由过程中，只有网络中的所有路由器的路由表完全一致了，该网络才能够正常转发数据。
若网络中的所有路由器的路由表完全一致后，该网络称之为【收敛】了。
只有收敛的网络才能够正常的转发数据
网络不收敛的状态称之为动荡状态｜抖动状态｜翻动状态

# 四、静态路由

- 静态路由是由管理员手动写在路由表中的，且为单向的。

```bash
<Huawei>system-view
[Huawei]ip route_static 172.16.1.0 24 10.1.1.2
```

## 特点

- 路由表是手工设置
- 除非网络管理员干预，否则静态路由不会发生变化
- 路由表的形成不需要占用网络资源
- 适用环境
  - 一般用于网络规模很小、拓扑结构固定的网络中

# 五、缺省路由

- 当路由器在自身的路由表中无法找到目标网络时，令路由器不要丢弃数据包，而是将数据包转发至一个默认的路由器上，这条路由器被称之为 —— 缺省路由

```bash
<Huawei>system-view
[Huawei]ip route-static 0.0.0.0 0 10.1.1.2
```

## 特点

- 在所有路由类型中，默认路由的优先级最低
- 适用环境
  - 一般应用在只有一个出口的末端网络【Stub Area】
  - 作为其它路由的补充出现 

# 六、路由器在转发数据包时的封装过程

- 路由器在路由数据包时，三层头部中的源IP地址与目标IP地址始终保持不变，而路由器会不断的使用自身接口的MAC地址做二层重封装，将数据包一路路由至目的地。
- `结论：MAC地址的传输仅具有本地效应`

# 七、单臂路由

- 交换机上的VLAN之间如何通信

## 实验配置

![单臂路由](../../images/HCIA/单臂路由/单臂路由.png)

**SW**

```bash
[SW]vlan ba 2 3
[SW-GigabitEthernet0/0/1]port link-type access
[SW-GigabitEthernet0/0/1]port default vlan 2
[SW-GigabitEthernet0/0/3]port link-type access
[SW-GigabitEthernet0/0/3]port default vlan 3
[SW-GigabitEthernet0/0/2]port link-type trunk

#放行vlan 2 3 标签
[SW-GigabitEthernet0/0/2]port trunk allow-pass vlan 2 to 3
```

**R1**

```bash
[R1]int g0/0/0.1

#dot1q termination vid
#首先删除VLAN标签。接口在收到VLAN报文后，剥掉报文中携带的Tag后进行三层转发。
#然后添加VLAN标签。接口在发送报文时，将相应的VLAN信息添加到报文中再发送。

[R1-GigabitEthernet0/0/0.1]dot1q termination vid 2
[R1-GigabitEthernet0/0/0.1]ip address 192.168.1.1 24
[R1-GigabitEthernet0/0/0.1]arp broadcast enable  #开启ARP广播功能

[R1-GigabitEthernet0/0/0.2]dot1q termination vid 3
[R1-GigabitEthernet0/0/0.2]ip address 192.168.2.1 24
[R1-GigabitEthernet0/0/0.2]arp broadcast enable
```

**测试连通性**

```bash
PC>ping 192.168.2.10

Ping 192.168.2.10: 32 data bytes, Press Ctrl_C to break
From 192.168.2.10: bytes=32 seq=1 ttl=127 time=63 ms
From 192.168.2.10: bytes=32 seq=2 ttl=127 time=78 ms
From 192.168.2.10: bytes=32 seq=3 ttl=127 time=94 ms
From 192.168.2.10: bytes=32 seq=4 ttl=127 time=78 ms
From 192.168.2.10: bytes=32 seq=5 ttl=127 time=78 ms

--- 192.168.2.10 ping statistics ---
  5 packet(s) transmitted
  5 packet(s) received
  0.00% packet loss
  round-trip min/avg/max = 63/78/94 ms
```