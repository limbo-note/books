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

- 如果目的主机和源主机在一个网路上，IP数据报就直接发送到对方；若不在，则发送给默认的路由器，由路由器来转发
- 每一个主机都可以配置成一个路由器
- 当从一个网络接口接收到数据报时，检查目的IP地址是否为本机IP或者广播地址：
	- 若是，则根据协议类型字段，对报文进行相应的处理
	- 若不是，则检查是否被设置了路由器的功能：
		- 若有路由器的功能，则进行转发（会对**路由表**进行搜索）
		- 若没有路由器的功能，则丢弃数据报

路由表包含以下字段：
- 目的IP地址。 可以是完整的主机地址（代表一个特定的主机），也可以是网络地址（代表一个网络）
- 下一站路由器的IP地址（若无路由器，则直接是下一个相连的网络IP地址）
- 标志。标明前两个字段到底是哪种IP地址
- 为数据报的传输指定的网络接口

IP路由选择规则：
- 搜索路由表，找到目的IP完全匹配的条目（优先搜索主机）
- 搜索路由表，找到目的IP网络号匹配的条目（其次匹配网络号）
- 搜索路由表，找到“默认”的条目（若以上都找不到，则选择默认）
- 若以上都找不到，则不发送数据报。（若数据报由本机产生，会返回一个“主机不可达”或“网络不可达”的错误）

两个例子：
- 同一以太网的两个主机的数据传送：				
	![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/3-3.jpg)
- 不同网络的主机的数据传送：						
	![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/3-4.jpg)
	- 大多数情况下使用的是默认路由
	- 数据报中的目的IP地址始终不变
	- 链路层的目的地址始终是下一站的链路层地址

### 3.4 子网寻址

A类和B类地址为主机号分配了太多的空间，一般会把（包括C类）主机号再分成一个子网号和一个主机号

- 子网的划分缩减了路由表的规模，如图：				
	![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/3-5.jpg)
	- 上图中所有的IP地址共有一个网络号140.252，而其内部有大量的子网
	- 外部路由器并不知道这些子网的划分，只需要知道目的地址为140.252开头的，都需要通过路由器gateway，所以只需要一个路由条目

### 3.5 子网掩码

子网掩码：确定有多少比特用于子网号，多少比特用于主机号。网络号和子网号的位置均为1，主机号的位置均为0

给定了**目的IP地址**和**子网掩码**后，再通过**本机IP**确定网络号和子网号的分界线，主机就可以确定目的地址类型：
- 本子网上的IP
- 本网络中其他子网的IP
- 其他网路的IP

### 3.6 特殊情况的IP地址

![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/3-6.jpg)

### 3.7 本书的子网

![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/3-7.jpg)
![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/3-8.jpg)

### 3.8 ifconfig命令
看后续章节慢慢补充

### 3.9 netstat命令
看后续章节慢慢补充

### 习题

1. 环回地址必须是127.0.0.1吗      
	
	任何网络号为127的A类地址都是环回地址，即`127.*.*.*`

2. 3.4节的图中有两个以上网络接口的路由器

	- kpno：3个点对点链路和2个以太网接口
	- R10：4个以太网接口
	- gateway：2个点对点链路和1个以太网接口
	- netb：1个以太网接口和2个点对点链路

3. 子网号为16 bit的A类地址与子网号为8 bit的B类地址的子网掩码有什么不同

	完全相同，均为255.255.255.0

4. RFC 1219，学习分配子网号和主机号的有关推荐技术

5. 子网掩码255.255.0.255是否对A类地址有效

	是合法的，用了非连续的子网掩码（即前8位和后8位作为子网掩码，中间8位作为主机号），但反对使用

# 4. ARP地址解析协议

- 地址解析协议ARP和RARP指的是将32位的IP地址和48位的以太网地址(mac地址)相互映射转化，如图：				
	
	![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/4-1.jpg)

### 4.2 FTP例子

在敲入命令`ftp [hostname]`后，会进行以下步骤：
1. 应用程序FTP调用`gethostbyname()`函数把主机名`hostname`转化成IP地址（即DNS解析）
2. 用此IP地址请求连接，完成TCP连接
3. 向目的地址发送IP数据报
4. 如果目的地址在本地网络（同一个网络）上，IP数据报可以直接发送到目的主机上；如果不在同一个网络上，则通过路由表发送到下一站的路由器
5. 若为以太网，需要将通过IP地址获得mac地址，即ARP协议的功能
6. ARP发送一份ARP请求的以太网数据帧给以太网的每个主机（即广播）。数据帧中包含目的IP地址，请求内容 “如果你是这个IP地址的拥有者，请回答你的硬件地址”
7. 目的主机收到ARP请求，会回应一个ARP应答（包含IP地址及对应的mac地址）
8. 收到ARP应答，获得mac地址
9. 发送IP数据报到目的主机

	![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/4-2.jpg)

- 注意：点对点链路不使用ARP，没有mac地址一说

### 4.3 ARP高速缓存

每个主机都有一个ARP高速缓存，存放了最近IP和mac地址的映射，生存时间一般为20分钟

可使用`arp -a`命令查看系统的ARP高速缓存

当系统收到ARP应答或发送ARP应答时，都会把映射记录存入高速缓存

### 4.4 ARP的分组格式

ARP格式如图：								
	![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/4-3.jpg)

### 4.5 ARP举例

- 一般的例子`bsdi% telnet svr4 discard`			
	 ![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/4-4.jpg)
- 对不存在的主机发送ARP请求
	- 因不会收到ARP应答，将多次超时后重新发送ARP请求，最后放弃请求
- ARP高速缓存超时设置
	- ARP高速缓存的条目中一般都要有超时值，完整的条目默认为20分钟，不完整的条目（即没有获取到对应的mac地址）默认为3分钟

### 4.6 ARP代理

![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/4-5.jpg)

gemini主机在进行了对sun主机的ARP请求后，gemini的映射表会如下（即sun主机和netb主机对应的mac地址相同）：	
	![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/4-6.jpg)

### 4.7 免费ARP

指主机发送ARP请求查找自己IP地址的mac地址，如图：	
	![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/4-7.jpg)

免费ARP的两个作用：
- 确定是否有另一个主机与本主机设置了相同的IP。正常设置的情况下，免费ARP不会有应答
- 通知以太网中的各主机更新高速缓存中的有关本主机条目（所有主机接收到广播的ARP请求都会进行高速缓存的更新）

### 4.8 arp命令

arp命令参数：
- `-a`，查询高速缓存的内容
- `-d`，删除高速缓存的某项内容
- `-s`，增加高速缓存的内容（默认是永久性的）,temp指定超时性，pub指定为ARP代理

### 习题

1. 执行命令`bsdi％ rsh svr4 arp -a`（在svr4主机上运行arp -a）前，目的主机的ARP高速缓存是空的。执行该命令后，发生什么

	引起在两个主机之间交换IP数据报，即使在执行rsh命令之前，svr4主机的ARP缓存是空的，当rsh服务器执行arp命令时，必须保证ARP缓存中有bsdi主机的相关条目

2. 如何判断主机是否能正确处理非必要的ARP请求

	判断主机A是否能正确处理。需要另一台主机B，首先保证主机A缓存中没有B相关的条目，然后手动在主机A缓存中加上一条temp选项的主机B的错误条目。并在主机B上发送一个免费ARP请求，观察主机A是否能够正确的更新缓存中的条目

3. 在ARP等待应答期间，如何处理在这期间收到相同目的IP地址发来的多个数据包

	见11.9节

4. 略

# 5. RARP逆地址解析协议

有磁盘的主机一般从配置文件中读取IP地址，**无磁盘**的主机就要先从接口卡中获取到硬件地址，然后通过RARP请求获取IP地址

### 5.2 RARP分组格式

分组格式与ARP基本一致，只有帧类型码、操作码不同；和ARP一样，RARP	请求也是广播的，应答也是单播的

### 5.3 RARP举例

sun主机广播RARP请求，由bsdi上的RARP服务程序响应：	
	![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/5-1.jpg)
	![](https://github.com/limbo-note/books/blob/master/TCPIP_ILLUSTRATED_I/5-2.jpg)

### 5.4 RARP服务器的设计

RARP服务的设计比较复杂（通常是用户进程），而ARP服务的设计很简单（通常是在内核中实现的）

- 作为用户进程的RARP服务
	- 一般要为多个主机（网络上的所有无盘系统）提供地址映射，映射保存在一个本地磁盘文件中。内核不读取此文件，所以一般都是用户进程来提供此服务
	- RARP服务的实现是与系统捆绑在一起的，因为要发送特殊类型的以太网数据帧，实现比较复杂
- 每个网络多个RARP服务
	- RARP请求是在硬件层上进行广播的，所以不会进行路由器转发。一个网络上一般都要提供多个RARP服务
	- 当多个服务都产生应答时，发送方选用最先达到的RARP请求

### 习题

1. RARP需要不同的帧类型字段吗，可以让ARP和RARP都使用相同的值吗

	理论上说不同的帧类型字段不是必须的，因为ARP和RARP分组的操作码都有一个不同的值（1~4）。但是RARP服务是用户进程，ARP服务是内核中实现的，设计成不同的帧类型方便处理

2. 当多个RARP服务都产生应答时，如何防止响应冲突

	- 不同的服务器设置不同的响应延迟，可避免冲突
	- 设置一个主服务器，其他是次服务器。次服务器只在主服务器关机情况下才工作