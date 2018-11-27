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
	- 用传输层和IP层的方法来处理环回数据，会经过传输层和IP层完整的处理过程，简化了设计

### 2.8 最大传输单元MTU

- 不同链路层协议有不同的MTU，MTU针对IP数据报的长度
- 如果IP数据报的长度比链路层的MTU大，那就IP层就会进行分片

### 2.9 路径MTU

- 路径MTU：两台主机通信时，要经过多个网络，每个网络都会有自己的MTU。 该路径中最小的网络MTU则是路径MTU
- 当选择不同的路由时，路径MTU就会不一样。所以从A到B的路由不一定等于从B到A的路由

### 2.10 串行线路吞吐量计算
### 习题

1.  netstat命令查看系统上的接口和MTU

	同3.9，`netstat -in`

# 3. IP网际协议

所有TCP/UDP/ICMP/IGMP都以IP数据报格式传输：
- IP协议是不可靠协议，不能保证数据报能成功到达目的
- IP协议是无连接的，并不维护关于后续数据报的状态信息。 IP数据报的处理相互独立，即可以不按发送顺序接收

### 3.2 IP首部

- IP数据报格式如下：							
	![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/3-1.jpg)
	- 目前协议版本号为4，即IPV4
	- 首部长度以4字节为单位，所以首部最长60个字节。IP首部长度必须为4字节的整数倍，不足补0
	- 不同应用程序有不同的推荐服务类型字段，如下图
		![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/3-2.jpg)
	- 总长度是整个IP数据报的长度，以字节为单位，所以IP数据报最长可达65535字节（会受到MTU的限制）。当数据报被分片时，该字段也会变化
	- TTL设置数据报可以经过的最多路由器数。初始值由源主机设置（一般为32或64），每经过一个路由器就减1。达到0时，该数据报就丢弃，并返回ICMP报文
	- 协议字段用于对IP数据报进行分用
	- IP的首部校验和只对IP首部进行计算
		- 校验过程：发送前，将校验和置0，对首部中每个16bit进行反码求和，结果即为校验和；接收时，同样对首部中每个16bit进行反码求和（包括了校验字段），结果正常应该为全1。若不为全1，则丢弃该数据报

### 3.3 IP路由选择

