# 1. 概述

### 1.2 分层

- 链路层（网络接口层）
	- 设备驱动和接口卡，处理与物理接口细节
	- 以太网协议
- 网络层
	- IP/ICMP/IGMP
	- 提供点到点的服务
- 运输层
	- TCP
		- 可靠性。应用层可忽略这些细节
	- UDP
		- 不可靠。可靠性可以由应用层提供
	- 提供端到端的服务 
- 应用层
	- Telnet远程登陆
	- FTP文件传输协议
	- SMTP简单邮件传送协议
	- SNMP简单网络管理协议
	- HTTP等									
	
![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/1-1.jpg)
![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/1-2.jpg)

IP协议是不可靠的，TCP才是可靠的。一个路由器具有两个或多个网络接口层（因为它连接了两个或多个网络）。

连接网络的另一个途径是使用网桥。网桥是在链路层上对网络进行互连，而路由器则是在网络层上对网络进行互连。网桥使得多个局域网（LAN）组合在一起，这样对上层来说就好像是一个局域网。

###  1.3 TCP/IP的分层

![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/1-3.jpg)

### 1.4 互联网的地址

- 五类IP地址如图：							
	![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/1-4.jpg)

互联网络信息中心只分配网络号，主机号由各系统的管理员自行分配。

按目的主机分类，三类IP地址：
- 单播地址：目的为单个主机
- 广播地址：目的端为给定网络上的所有主机
- 多播地址：目的端为同一组内的所有主机

### 1.5 域名系统DNS
任何应用程序都可以调用一个标准的库函数来查看给定名字的主机的IP地址。类似地，系统还提供一个逆函数—给定主机的IP地址，查看它所对应的主机名

### 1.6 封装

![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/1-5.jpg)

### 1.7 分用

从协议栈中由底向上升，去掉各层协议的报文首部，同时确定上层协议类型，这叫做**分用**，过程如图：		
![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/1-6.jpg)

协议ICMP/IGMP和IP协议是同一层，但是都被封装在IP数据报中

### 1.8 客户服务器模型

- 重复型
	1. 等待一个客户请求的到来
	2. 处理客户请求
	3. 发送响应给发送请求的客户
	4. 返回1步
	- 问题在2状态，这个时候，它不能为其他客户机提供服务
- 并发型
	1. 等待一个客户请求的到来
	2. 启动一个新的服务器（进程、线程、任务）来处理这个客户的请求。处理结束后，终止这个新的服务
	3. 返回1步
	- 优点在于可并发处理
- 一般来说，TCP服务器是并发的，而UDP服务器是重复的

### 1.12 标准的简单服务

- 一些常用的标准的服务程序：			
	![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/1-7.jpg)
- 客户端常用的程序选择Telnet

### 1.16 本书的测试网络环境
![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/1-8.jpg)

### 习题

1. 最多有多少个 A类、 B类和C类网络号

	(2^7-2)＋(2^14-2)＋(2^21-2) = 2 113 658，全0或全1网络号是非法的

2. NSFNET网络上登记的网络数的变化趋势

	![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/1-9.jpg)

# 2. 链路层

链路层主要目的有三个：
- 为IP模块发送和接收IP数据报
- 为ARP模块发送ARP请求和接收ARP应答
- 为RARP模块发送RARP请求和接收RARP应答

不同的链路层协议：**以太网**、令牌环网、FDDI等，及串行接口链路层协议**SLIP**和PPP等

### 2.2 以太网和IEEE802封装

![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/2-1.jpg)

### 2.4 SLIP串行线路IP

- SLIP协议定义的格式：						
	![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/2-2.jpg)
- 缺陷
	- 无IP地址信息
	- 没有类型字段，线路不能同时使用其它协议
	- 没有校验和字段

### 2.6 PPP点对点协议

- PPP协议格式：								
	![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/2-3.jpg)
- 相比SLIP，有很多优点
	- 支持串行线路上的多种协议，不只是IP协议
	- 有校验和字段
	- 有IP地址信息
	- 对TCP/IP报文首部进行压缩
	- 可对链路选项进行设置

### 2.7 环回接口

- 用以允许运行在同一台主机上的客户程序和服务器程序通过TCP/IP进行通信，大多数系统把127.0.0.1这个IP分配给此接口，并命名为localhost

- 环回接口处理IP数据报的简单过程：					
	![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/2-4.jpg)