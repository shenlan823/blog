---
title: HCIA -- 14 -- NAT
date: 2023-09-08 21:55:55
tags:
  - HCIA
  - NAT
---
# 一、NAT的技术背景

1. IPv4地址日益紧缺，过度至IPv6需要花费大量时间，因此需要一些过渡性技术。
2. 内网用户使用内部私有地址进行互联，但是私有地址不会被ISP的设备路由，无法访问Internet。
3. 公司从ISP租用一个固定的公有IP地址，将该IP地址配置在路由器的外网接口上，当内部用户想要访问Internet时，路由器负责将私有地址转换为外网的公有IP地址，实现地址转化的效果。
4. NAT具有很好的隐藏内部私有IP地址的作用，包含内网安全性。
5. NAT具有端口映射的能力，能够将内部服务器的IP及端口映射在外网接口上，从而实现内网服务器向外网用户提供服务的功能。

# 二、NAT的工作原理

1. NAT与ACL一样，都关注第三层【IP头部】与第四层【TCP/UDP头部】
2. NAT在转换时，只转换IP地址，不会转换端口号码。

`注：路由器在提供NAT服务时，会使用自身借口的IP地址替换用户的源IP地址，但是不会替换和更改端口号码；路由器要依靠不同的端口号码来区分不同的响应报文该回复给哪个客户。`

# 三、网络地址转换的方式

1. 静态转换
2. 动态转换
3. Easy IP
4. NATP【端口多路复用】

# 四、NAT的四个关键性术语

1. 内部局部地址【Inside Local】：私有地址
2. 外部局部地址【Outside Local】：公有地址
3. 内部全局地址【Inside Global】：公有地址
4. 外部全局地址【Outside Global】：公有地址

# 五、NAT的优缺点

1. 优点：
   1. 能够在最大限度上节约公有IP地址
   2. 能够处理地址交叉的问题
   3. 部署灵活
   4. 通过转换，很好的隐藏了内部私有地址，保护内网安全
2. 缺点：
   1. 若内部私有地址过多，同时通过1个公有地址访问外网，内部网络容易出现延迟增大的问题
   2. 配置较为复杂
   3. 某些服务不支持

# 六、实验配置

![NAT.png](../../images/HCIA/NAT/NAT.png)

## 静态转换

**R1**

```bash
[AR1]dis ip int br
······
Interface                         IP Address/Mask      Physical   Protocol  
GigabitEthernet0/0/0              192.168.1.1/24       up         up        
GigabitEthernet0/0/1              10.1.1.1/24          up         up
······

[AR1-GigabitEthernet0/0/1]nat static global 10.1.1.10 inside 192.168.1.10
[AR1-GigabitEthernet0/0/1]nat static global 10.1.1.20 inside 192.168.1.20
[AR1-GigabitEthernet0/0/1]nat static global 10.1.1.30 inside 192.168.1.30
[AR1-GigabitEthernet0/0/1]nat static enable 
```

```bash
[AR1]dis nat static 
  Static Nat Information:
  Interface  : GigabitEthernet0/0/1
    Global IP/Port     : 10.1.1.10/---- 
    Inside IP/Port     : 192.168.1.10/----
    Protocol : ----     
    VPN instance-name  : ----                            
    Acl number         : ----
    Netmask  : 255.255.255.255 
    Description : ----

    Global IP/Port     : 10.1.1.20/---- 
    Inside IP/Port     : 192.168.1.20/----
    Protocol : ----     
    VPN instance-name  : ----                            
    Acl number         : ----
    Netmask  : 255.255.255.255 
    Description : ----

    Global IP/Port     : 10.1.1.30/---- 
    Inside IP/Port     : 192.168.1.30/----
    Protocol : ----     
    VPN instance-name  : ----                            
    Acl number         : ----
    Netmask  : 255.255.255.255 
    Description : ----

  Total :    3
```

```bash
PC>ping 10.1.1.2

Ping 10.1.1.2: 32 data bytes, Press Ctrl_C to break
From 10.1.1.2: bytes=32 seq=1 ttl=254 time=79 ms
From 10.1.1.2: bytes=32 seq=2 ttl=254 time=46 ms
From 10.1.1.2: bytes=32 seq=3 ttl=254 time=47 ms
From 10.1.1.2: bytes=32 seq=4 ttl=254 time=32 ms
From 10.1.1.2: bytes=32 seq=5 ttl=254 time=46 ms

--- 10.1.1.2 ping statistics ---
  5 packet(s) transmitted
  5 packet(s) received
  0.00% packet loss
  round-trip min/avg/max = 32/50/79 ms
```

## 动态转换

**R1**

```bash
[AR1]nat address-group 1    10.1.1.10 	   10.1.1.100
# 			  Start address   End address
#创建地址池

[AR1]acl 2000
#通过 ACL 匹配需要被转换的网段

[AR1-acl-basic-2000]rule permit source 192.168.1.0 0.0.0.255
[AR1-acl-basic-2000]rule deny source any
#只转换 192.168.1.0/24 网段的地址，不转换其他的地址。

[AR1-GigabitEthernet0/0/1]nat outbound 2000 address-group 1 #no-pat
#no-pat不转换端口
```

```bash
PC>ping 10.1.1.2

Ping 10.1.1.2: 32 data bytes, Press Ctrl_C to break
From 10.1.1.2: bytes=32 seq=1 ttl=254 time=63 ms
From 10.1.1.2: bytes=32 seq=2 ttl=254 time=31 ms
From 10.1.1.2: bytes=32 seq=3 ttl=254 time=31 ms
From 10.1.1.2: bytes=32 seq=4 ttl=254 time=63 ms
From 10.1.1.2: bytes=32 seq=5 ttl=254 time=47 ms

--- 10.1.1.2 ping statistics ---
  5 packet(s) transmitted
  5 packet(s) received
  0.00% packet loss
  round-trip min/avg/max = 31/47/63 ms

```

```bash
#查看地址池
[AR1]dis nat address-group 1

 NAT Address-Group Information:
 --------------------------------------
 Index   Start-address      End-address
 --------------------------------------
 1           10.1.1.10       10.1.1.100
 --------------------------------------
  Total : 1
```

```bash
[AR1]dis nat session all
NAT Session Table Information:

     Protocol          : ICMP(1)
     SrcAddr   Vpn     : 192.168.1.20                                   
     DestAddr  Vpn     : 10.1.1.2                                       
     Type Code IcmpId  : 0   8   52680
     NAT-Info
       New SrcAddr     : 10.1.1.21      
       New DestAddr    : ----
       New IcmpId      : 10247

     Protocol          : ICMP(1)
     SrcAddr   Vpn     : 192.168.1.10                                   
     DestAddr  Vpn     : 10.1.1.2                                       
     Type Code IcmpId  : 0   8   52673
     NAT-Info
       New SrcAddr     : 10.1.1.11      
       New DestAddr    : ----
       New IcmpId      : 10245

     Protocol          : ICMP(1)
     SrcAddr   Vpn     : 192.168.1.30                                   
     DestAddr  Vpn     : 10.1.1.2                                       
     Type Code IcmpId  : 0   8   52685
     NAT-Info
       New SrcAddr     : 10.1.1.31      
       New DestAddr    : ----
       New IcmpId      : 10249

······

  Total : 15
```

## Easy IP

**R1**

```bash
#Easy IP配置与动态转换的配置方式几乎一样

[AR1]acl 2000
#通过 ACL 匹配感兴趣流量

[AR1-acl-basic-2000]rule permit source 192.168.1.0 0.0.0.255
[AR1-acl-basic-2000]rule deny source any

#不需要创建地址池

[AR1-GigabitEthernet0/0/0]nat outbound 2000
```

```bash
PC>ping 10.1.1.2

Ping 10.1.1.2: 32 data bytes, Press Ctrl_C to break
From 10.1.1.2: bytes=32 seq=1 ttl=254 time=47 ms
From 10.1.1.2: bytes=32 seq=2 ttl=254 time=31 ms
From 10.1.1.2: bytes=32 seq=3 ttl=254 time=47 ms
From 10.1.1.2: bytes=32 seq=4 ttl=254 time=31 ms
From 10.1.1.2: bytes=32 seq=5 ttl=254 time=47 ms

--- 10.1.1.2 ping statistics ---
  5 packet(s) transmitted
  5 packet(s) received
  0.00% packet loss
  round-trip min/avg/max = 31/40/47 ms
```