# 1. 概述

### 1.2 分层

- 链路层（网络接口层）
	- 设备驱动和接口卡，处理与物理接口细节
	- 以太网协议
- 网络层
	- IP/ICMP/IGMP
- 运输层
	- TCP
		- 可靠性。应用层可忽略这些细节
	- UDP
		- 不可靠。可靠性可以由应用层提供
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