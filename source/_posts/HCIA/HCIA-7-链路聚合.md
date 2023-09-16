---
title: HCIA -- 7 -- 聚合链路
date: 2023-08-31 22:59:05
tags:
  - HCIA
  - 聚合链路
---


## 应用场景

- 链路聚合大多部署在核心层，以便提升整体网络的数据吞吐量。

## 特点

- 链路聚合能够提高链路带宽、增强网络可用性、支持负载分担

## 模式

- 华为的网络设备共有两种链路聚合模式：
  - 手工负载分担模式：所有活动端口均可参与数据的转发，负载分担流量
  - LACP【Link Aggregation Control Protocol】模式：支持链路备份
![模式](../../images/HCIA/链路聚合/模式.png)

## 指导原则

1. 华为网络设备最多允许 **8个端口** 绑定刀一起
2. 一个 **Eth-Trunk** 内的所有端口必须使用 **相同模式**【手工或LACP】
3. 一个 **Eth-Trunk** 内的所有端口都必须具有 **相同的速率和双工模式**；若成员端口的速率不同，速率较低的端口可能会拥塞，报文可能会被丢弃
4. 一个端口不能在 **相同时间** 内属于 **多个通道组**
5. 一个 **Eth-Trunk** 内的所有端口都必须配置到 **相同的接入VLAN或VLAN干道中**
6. 只能删除不包含任何成员端口的 Eth-Trunk 口
7. 将端口加入进 Eth-Trunk 口时，二层 Eth-Trunk 口的成员端口必须是二层端口，三层 Eth-Trunk 口的成员端口必须时三层端口
8. 加入 Eth-Trunk 口的端口必须是 **hybrid端口**
9. 一个 Eth-Trunk 口不能充当其它 Eth-Trunk 口的成员端口
10. 一个 Eth-Trunk 口的成员端口类型必须相同；一个快速以太网端口【FastEthernet】与一个千兆以太网端口【GigabitEthernet】不能加入同一个 Eth-Trunk 内
11. 端口加入到 Eth-Trunk 后，Eth-Trunk 端口将会**学习** `MAC地址`，而成员端口不在学习 `MAC地址`

`注：二层交换机只能和二层交换机做链路聚合，三层交换机只能和三层交换机做链路聚合。`

## 数据流控制

- 数据流在聚合链路上传输，数据顺序必须保持不变。一个数据流可以看做是一组 MAC地址 和 IP地址 相同的帧。
- 配置链路聚合后，多条物理链路被绑定成一条聚合链路，一个数据流中的帧通过不同的物理链路传输，从而产生接受数据包乱序的情况。
- 为了避免这种情况的发生，Eth-Trunk 采用逐流负载分担的机制
- 将数据帧中的地址通过 HASH 算法生成 HASH-KEY 值，根据该数值在 Eth-Trunk 转发表中寻找对应的出接口，不同的 MAC地址 或 IP地址 HASH 得出的 HASH-KEY 值不同，从而出接口也就不同。
- 保证了同一数据流的帧在同一条物理链路转发，又实现了流量在聚合组内各物理链路上的负载分担，即逐流的负载分担。
- 逐流负载分担能保证包的顺序，但不能保证贷款利用率。

## 均分负载

- 等价均分负载 — 将流量均匀地分布到多条度量相同的路径上，又叫负载平衡。
- 非等价均分负载 — 将数据包分布到度量不同的多条路径上，各条路径上分布的流量与路由代价成反比，代价越低的路径分配的流量越多，代价越高的路径分配的流量越少。
- 一些路由选择协议可以支持等价和非等价负载均衡两种方式，如 EIGRP；而一些路由选择协议仅支持等价方式，静态路由没有度量，因此仅支持等价负载均衡。

# 实验配置

## 二层链路手工模式聚合配置

![二层-链路聚合](../../images/HCIA/链路聚合/二层-聚合链路.png)

**SW1**

```shell
[SW1]int Eth-Trunk 1
[SW1-Eth-Trunk1]q
[SW1]int e0/0/1
[SW1-Ethernet0/0/1]eth-trunk 1
[SW1-Ethernet0/0/1]int e0/0/2
[SW1-Ethernet0/0/2]eth-trunk 1
[SW1-Ethernet0/0/2]int e0/0/3
[SW1-Ethernet0/0/3]eth-trunk 1

[SW1]dis ip int br
*down: administratively down
^down: standby
(l): loopback
(s): spoofing
The number of interface that is UP in Physical is 2
The number of interface that is DOWN in Physical is 1
The number of interface that is UP in Protocol is 1
The number of interface that is DOWN in Protocol is 2
Interface                         IP Address/Mask      Physical   Protocol  
MEth0/0/1                         unassigned           down       down      
NULL0                             unassigned           up         up(s)     
Vlanif1                           unassigned           up         down      

[SW1]dis eth-trunk 1
Eth-Trunk1's state information is:
WorkingMode: NORMAL         Hash arithmetic: According to SIP-XOR-DIP         
Least Active-linknumber: 1  Max Bandwidth-affected-linknumber: 8              
Operate status: up          Number Of Up Port In Trunk: 3                     
--------------------------------------------------------------------------
PortName                      Status      Weight 
Ethernet0/0/1                 Up          1      
Ethernet0/0/2                 Up          1      
Ethernet0/0/3                 Up          1   



[SW1]dis stp br
 MSTID  Port                        Role  STP State     Protection
   0    Ethernet0/0/4               DESI  FORWARDING      NONE
   0    Eth-Trunk1                  ROOT  FORWARDING      NONE
```

**SW2**
sw2配置与sw1相同

```bash
[SW2]int Eth-Trunk 1
[SW2-Eth-Trunk1]int e0/0/1
[SW2-Ethernet0/0/1]eth-trunk 1
[SW2-Ethernet0/0/1]int e0/0/2
[SW2-Ethernet0/0/2]eth-trunk 1
[SW2-Ethernet0/0/2]int e0/0/3
[SW2-Ethernet0/0/3]eth-trunk 1
```

## LACP

**SW1**

```bash
[SW1]int Eth-Trunk 1
[SW1-Eth-Trunk1]mode lacp-static  #配置模式
[SW1-Ethernet0/0/1]eth-trunk 1
[SW1-Ethernet0/0/1]int e0/0/2
[SW1-Ethernet0/0/2]eth-trunk 1
[SW1-Ethernet0/0/2]int e0/0/3
[SW1-Ethernet0/0/3]eth-trunk 1
[SW1]lacp priority 100  #选举指定lacp优先级
[SW1]int e0/0/1
[SW1-Ethernet0/0/1]lacp priority 100  #配置优先级
[SW1-Ethernet0/0/1]int e0/0/2
[SW1-Ethernet0/0/2]lacp priority 100
[SW1]int Eth-Trunk 1
[SW1-Eth-Trunk1]max active-linknumber 2  #指定最大活跃链路为两条

[SW1]dis eth-trunk 1
Eth-Trunk1's state information is:
Local:
LAG ID: 1                   WorkingMode: STATIC                               
Preempt Delay: Disabled     Hash arithmetic: According to SIP-XOR-DIP         
System Priority: 100        System ID: 4c1f-cc82-4664                         
Least Active-linknumber: 1  Max Active-linknumber: 2                          
Operate status: down        Number Of Up Port In Trunk: 0                     
--------------------------------------------------------------------------------
ActorPortName          Status   PortType PortPri PortNo PortKey PortState Weight
Ethernet0/0/1          Unselect 100M     100     2      289     10100010  1     
Ethernet0/0/2          Unselect 100M     100     3      289     10100010  1     
Ethernet0/0/3          Unselect 100M     32768   4      289     10100010  1     

Partner:
--------------------------------------------------------------------------------
ActorPortName          SysPri   SystemID        PortPri PortNo PortKey PortState
Ethernet0/0/1          0        0000-0000-0000  0       0      0       10100011
Ethernet0/0/2          0        0000-0000-0000  0       0      0       10100011
Ethernet0/0/3          0        0000-0000-0000  0       0      0       10100011
'
[SW1]dis stp br
 MSTID  Port                        Role  STP State     Protection
   0    Ethernet0/0/4               DESI  FORWARDING      NONE
   0    Eth-Trunk1                  ROOT  FORWARDING      NONE
```

**SW2**
sw2配置与sw1相同

```bash
[SW2]int Eth-Trunk 1
[SW2-Eth-Trunk1]mode lacp-static
[SW2-Eth-Trunk1]int e0/0/1
[SW2-Ethernet0/0/1]eth-trunk 1
[SW2-Ethernet0/0/1]int e0/0/2
[SW2-Ethernet0/0/2]eth-trunk 1
[SW2-Ethernet0/0/2]int e0/0/3
[SW2-Ethernet0/0/3]eth-trunk 1
[SW2]lacp priority 100
[SW2]int e0/0/1
[SW2-Ethernet0/0/1]lacp priority 100
[SW2-Ethernet0/0/1]int e0/0/2
[SW2-Ethernet0/0/2]lacp priority 100
[SW2]int Eth-Trunk 1
[SW2-Eth-Trunk1]max active-linknumber 2
```

## 三层链路手工模式聚合配置

![三层-链路聚合](../../images/HCIA/链路聚合/三层-聚合链路.png)

**AR1**

```bash
[AR1]int Eth-Trunk 1
[AR1] int g0/0/0
[AR1-GigabitEthernet0/0/0]eth-trunk 1
Info: This operation may take a few seconds. Please wait for a moment...
Error: The layer-3 interface can not add into a layer-2 trunk.done.
[AR1]int Eth-Trunk 1
[AR1-Eth-Trunk1]undo portswitch  #关闭二层端口功能
[AR1-Eth-Trunk1]ip address 192.168.1.1 24
[AR1-GigabitEthernet0/0/0]eth-trunk 1
[AR1-GigabitEthernet0/0/0]int g0/0/1
[AR1-GigabitEthernet0/0/1]eth-trunk 1

#测试连通性
[AR1]ping 192.168.1.2
  PING 192.168.1.2: 56  data bytes, press CTRL_C to break
    Reply from 192.168.1.2: bytes=56 Sequence=1 ttl=255 time=90 ms
    Reply from 192.168.1.2: bytes=56 Sequence=2 ttl=255 time=10 ms
    Reply from 192.168.1.2: bytes=56 Sequence=3 ttl=255 time=20 ms
    Reply from 192.168.1.2: bytes=56 Sequence=4 ttl=255 time=30 ms
    Reply from 192.168.1.2: bytes=56 Sequence=5 ttl=255 time=10 ms

  --- 192.168.1.2 ping statistics ---
    5 packet(s) transmitted
    5 packet(s) received
    0.00% packet loss
    round-trip min/avg/max = 10/32/90 ms

[AR1]int Eth-Trunk 1
[AR1-Eth-Trunk1]mode lacp-static 

[AR1]dis int Eth-Trunk 1
Eth-Trunk1 current state : UP
Line protocol current state : UP
······
-----------------------------------------------------
PortName                      Status      Weight
-----------------------------------------------------
GigabitEthernet0/0/0          UP          1
GigabitEthernet0/0/1          UP          1
-----------------------------------------------------
The Number of Ports in Trunk : 2
The Number of UP Ports in Trunk : 2
```

**AR2**
AR2配置与AR1相同

```shell
[AR2]int Eth-Trunk 1
[AR2-Eth-Trunk1]undo portswitch
[AR2-Eth-Trunk1]ip address 192.168.1.2
[AR2-GigabitEthernet0/0/0]eth-trunk 1
[AR2-GigabitEthernet0/0/0]int g0/0/1
[AR2-GigabitEthernet0/0/1]eth-trunk 1
```