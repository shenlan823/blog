---
title: HCIA -- 15 -- PPP
date: 2023-09-09 22:53:16
tags:
  - HCIA
  - PPP
---
# 一、PPP的基本概念

1. PPP【Point to Point Protocol】是属于数据链路层协议，不会因IP的问题导致down掉，PPP的连接建立完全依靠数据链路层的MAC
2. PPP的前身SLTP【Serial Line Internet Protocol｜串行链路互联网协议】
3. SLIP的缺点：
   1. SLIP不支持地址自动协商，需要用户自行配置
   2. SLIP只支持IP协议，在其数据包中，不包含协议字段，因此其无法支持IPX、Appletalk、MPLS
   3. SLIP不支持校验功能，无法对传输的数据进行校验和检测
   4. 1989年，PPP被正式提出，1994年成为互联网标准协议之一

# 二、PPP的优势

1. PPP同时支持同步传输与异步传输通信
2. 支持各种各样的网络层协议【IP、IPX、AppleTalk、MPLS】
3. 支持地址的自动协商
4. 支持数据的校验和检测
5. 支持PAP、CHAP这两种用户验证
6. 支持数据压缩【仅对有效的载荷数据进行压缩】

# 三、PPP的组成

![PPP的组成.png](../../images/HCIA/PPP/PPP的组成.png)

1. PPP是由3个部分组成的
   1. 物理层的封装报文的方式【同步传输/异步传输】
   2. LCP【Link Control-Protocol｜链路控制协议】：用来建立、维护、配置、终止一段数据链路层的协议
   3. NCP【Network Control-Protocol｜网络控制协议】：用来在一个已经建立完连接的PPP网络上进行IP层的协商与通讯

# 四、PPP链路的建立

![PPP链路的建立](../../images/HCIA/PPP/PPP链路的建立.png)

1. 物理链路不可用阶段
2. 数据链路层的链路建立阶段
3. 提交用户认证阶段
4. 认证成功进入网络协商阶段
5. 若认证失败则进入链路终止阶段

# 五、PPP的帧格式

![PPP的帧格式](../../images/HCIA/PPP/PPP的帧格式.png)

# 六、PPP支持2种认证

1. PAP【Password Authentication Protocol｜口令认证协议】
   - PAP是2次握手认证协议
   - 由被认证方主动发起认证请求
   - 所有的用户名及密钥均使用明文方式传递
2. CHAP【Challenge HandShake Authentication Protocol｜质询握手认证协议】
   - CHAP时3次握手认证协议
   - 由主认证方首先发起认证请求
   - 主/被双方相互验证
   - 密钥并不直接发送，而是使用三次握手过程中保持不变的ID值，主认证方随机生成的数据，以及双方共同知道的密钥，合在一起，使用HASH的MD5进行加密，加密之后再发送

# 实验配置

## PAP

## CHAP
