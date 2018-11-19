# 1. 深入Web请求过程

B/S架构好处：
- 客户端浏览器统一，对用户友好
- 服务端基于统一的HTTP/HTTPS

### 1.1 B/S网络架构概述

C/S大多采用长连接，B/S的HTTP则是无状态的短连接。大多数B/S架构如下图：

![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/1-1.png)

根据上图，首先请求DNS把URL解析成IP，找到对应服务器并发送get请求，服务器返回资源。其中细节的业务逻辑比较复杂：服务器有多台时，需要一个负载均衡器分配请求；请求的数据可能在分布式缓存或静态文件或数据库中；当浏览器解析数据时可能还会发现一些静态资源需要另外发起HTTP请求

网络结构需要固定的原则需要遵守：
- 互联网的所有资源都要用一个URL来表示
- 必须基于HTTP/HTTPS与服务端交互
- 数据展示必须在浏览器中进行

### 1.2 如何发起一个请求
建立HTTP请求其实就是建立Socket连接，只是发送的数据要符合HTTP的格式。 所以可以使用java中的HttpClient包模拟发包，Linux中可以使用curl命令发包

### 1.3 HTTP解析
请求头、响应头、状态码

- 浏览器缓存机制
	- Ctrl+F5刷新页面，请求的一定是最新的页面
	- 首先，Ctrl+F5会使浏览器端发送请求，而不是用本地的缓存数据
	- 其次，Ctrl+F5会在请求头中加一些请求头，告诉服务端要获取最新的数据而不是服务器中缓存的数据
	- 增加的请求头如`Cache-Control`、`Pragma`、`Expires`、`Last-Modified/Etag`

### 1.4 DNS域名解析
目前世界上的整个互联网有几个DNS根域名服务器，任何一个坏掉都会后果非常严重

域名解析过程如图：  

![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/1-2.png)

在浏览器中输入域名按回车后，DNS解析有十个步骤：
1. 浏览器检查缓存中是否存在，若存在则结束。缓存的大小和时间都有限制
2. 若浏览器缓存中没有，浏览器会查找操作系统缓存中是否有（windows中在C:\Windows\System32\drivers\etc\hosts设置），例如在测试时可以把域名解析到一台测试服务器上，Linux是在/etc/hosts中
3. 前两步在本机完成，若本机解析不到。开始请求域名服务器，一般网络配置中都有DNS服务器这项。windows下可通过ipconfig查看，Linux下在/etc/resolv.conf中配置。一般是本地区的域名服务器(LDNS)，一般都能请求到结果
4. 若上述步骤仍然不命中，则请求根域名服务器(Root Server)
5. 根域名服务器返回给本地区域名服务器(LDNS)一个主域名服务器（国际顶级域名服务器(gTLD)）
6. LDNS向gTLD发送请求
7. gTLD查找并返回此域名对应的Name Server服务器地址
8. Name Server查找IP，连同TTL值返回给LDNS
9. LDNS缓存此域名和IP，缓存时间由TTL控制
10. LDNS把结果返回给用户，用户缓存在本地

- linux/windows都可用`nslookup`查询域名解析结果
- linux中可用`dig`查询解析过程
- 清除缓存的域名
	- 解析结果会在LDNS和本地机器中缓存
	- LDNS的缓存时间就是TTL控制的，个人很难人工介入
	- 本地的缓存可通过`ipconfig/flushdns`（windows）或`/etc/init.d/nscd restart`（linux）来清除
	- 注意JVM中也可能会缓存DNS解析结果
- 几种域名解析方式
	- A记录：A代表Address，指定域名对应的IP（如：item.taobao.com对应115.238.23.xxx）。 A记录可以多个域名解析到一个IP，但不能将一个域名解析到多个IP
	- MX记录：Mail Exchange. （如：taobao.com的A记录是115.238.23.xxx，再将MX记录设为115.238.25.xxx，此时发送邮件至@taobao.com，就会发送至238.25的服务器，而正常的web请求还是得到A记录中的238.23服务器）
	- CNAME记录：别名解析。可以为一个域名设置多个别名
	- NS记录：为某个域名指定DNS解析服务器
	- TXT记录：为某个主机名或域名设置说明，类似注释

### 1.5 CDN工作机制

内容分布网络(Content Delivery Network)，目前CDN都以缓存网站中的静态数据为主，如CSS，JS，图片和静态页面等数据。用户从主站服务器获取动态内容后，再从CDN下载对应的静态数据，加速下载速度

架构如图：

![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/1-3.png)

- 负载均衡
	- 链路负载均衡。通过DNS解析成不同的IP，用户根据不同的IP来访问不同的服务器![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/1-4.png)
	- 集群负载均衡。又分为硬件、软件负载均衡，硬件负载均衡用一台昂贵的、性能好的服务器转发请求；软件负载均衡成本低，但一次访问请求要经过多次代理服务器，增加网络延迟![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/1-5.png) ![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/1-6.png)
	- 操作系统负载均衡。利用操作系统级别的软中断或硬中断达到负载均衡
- CND动态加速 
	- 在CDN的DNS解析中通过动态的链路探测寻找回源最好的一条路径

# 2. 深入分析Java I/O工作机制

### 2.1 Java的I/O类库的基本架构
java.io下大概有80个类，大概可分为四组：
- 基于字节：InputStream、OutputStream
- 基于字符：Writer、Reader
- 基于磁盘：File
- 基于网络：Socket

前两种关于**数据格式（以什么方式读写）**，后两种关于**传输方式（读写到哪里）**；这是影响效率的两大关键因素

- 基于字节的I/O
	- 操作数据的方式是可以组合使用的（即嵌套使用）
	- 必须指定流最终写到什么地方，要么磁盘要么网络
- 基于字符的I/O
	- I/O本质操作的都是字节不是字符，字符只是为了操作方便
	- Writer和Reader都只是定义了读写数据字符的方式，没有规定到要写到哪里
- 字节和字符的转化接口
	- 读写磁盘和网络传输都是基于字节的，必须有字符和字节的转换
	- 字节到字符的转换基于InputStreamReader类，如FileReader（继承自InputStreamReader）；字符到字节的转换基于OutputStreamWriter类，如FileWriter（继承自OutputStreamWriter）

### 2.2 磁盘I/O工作机制

- 访问文件的方式（应用程序访问物理设备只能通过系统调用，读写分别对应read()和write()两个系统调用）
	- 标准访问文件的方式。程序调用read()时，操作系统检查缓存是否存在，若存在则直接返回，如不存在，则从磁盘中读；同理，调用write()时，将数据从用户地址空间复制到内核地址空间的缓存，这时对程序来说已经写完成，写到磁盘的时机由操作系统决定或显式调用同步命令![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/2-1.jpg)
	- 直接I/O的方式。直接访问磁盘，不经过内核缓存![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/2-2.jpg)
	- 同步访问文件的方式。当数据成功写入磁盘时才返回，性能一般比较差（比标准方式还同步）
	- 异步访问文件的方式。线程发出读写请求后，去执行别的任务，直到请求的数据返回才继续（比标准方式还异步）
	- 内存映射的方式。将应用缓存和内核缓存关联起来，减少内核空间到 用户空间的复制![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/2-3.jpg)
- Java访问磁盘文件
	- 数据在磁盘中的唯一最小描述就是文件，上层程序只能通过文件来操作磁盘数据。文件也是操作系统和磁盘驱动器交互的最小单元
	- 以File类为例，与磁盘交互过程如图![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/2-4.jpg)
- Java序列化技术（将对象转化成二进制的字节数组）
	- 继承Serializable接口实现序列化
	- 序列化出来的二进制数据包含五部分：序列化文件头、序列化的类的描述、各个域的描述、父类信息描述、域的实际值（若域是对象，还将序列化这个对象）
	- 序列化的复杂情况包括：![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/2-5.jpg)
	- Java自带的序列化在跨平台时可能产生各种错误，尽量用通用的数据结构如JSON或XML

### 2.3 网络I/O工作机制

- TCP状态转化
	
	![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/2-6.jpg)
	- CLOSED. 起点、超时或连接关闭时的状态
	- LISTEN. 服务端等待连接，一般调用Socket、bind、listen
	- SYN-SENT. 客户端发起连接，发送SYN。若服务端不能连接，则直接进入CLOSED
	- SYN-RCVD. 服务端接受客户端的SYN请求，由LISTEN进入SYN-RCVD，同时服务端回应一个ACK（第一次），发送一个SYN给客户端；或者，客户端在发送SYN后接收到服务端的SYN请求，回应ACK（第二次），由SYN-SENT进入SYN-RCVD
	- ESTABLISHED. 服务端收到ACK（第三次），握手完成，建立连接
	- FIN-WAIT1. 主动关闭的一方，主动发送FIN（或两边同时关闭，同时发送FIN），由ESTABLISHED进入FIN-WAIT1
	- FIN-WAIT2. 主动关闭的一方，收到FIN ACK，由FIN-WAIT1进入FIN-WAIT2，不再接收数据，但是能发送数据
	- CLOSE-WAIT. 被动关闭的一方，接受到FIN后，回应FIN ACK，由ESTABLISHED进入CLOSE-WAIT
	- LAST-ACK. 被动关闭的一方，发送FIN，由CLOSE-WAIT进入LAST-ACK
	- CLOSING. 两边同时关闭时，收到对方的FIN时，回应FIN ACK，由FIN-WAIT1进入CLOSING
	- TIME-WAIT. 由FIN-WAIT2进入TIME-WAIT，主动关闭的一方收到被动关闭的一方的FIN；由CLOSING进入TIME-WAIT，两边同时关闭时，收到了对方的FIN ACK；由FIN-WAIT1进入TIME-WAIT，同时收到FIN（对方发起）和ACK（本身的FIN回应），与CLOSING转换到TIME-WAIT的区别ACK先于对方的FIN，若是FIN先于ACK，则是CLOSING转换到TIME-WAIT

- 影响网络传输的因素
	- 网络带宽。物理链路1S内能够传输的最大bit（位）数
	- 传输距离。主要影响传输延时
	- TCP拥塞控制
- Java Socket工作机制（与其它Socket一样）
- 建立通信链路
	- 套接字包含本地IP、端口，远程IP、端口
	- 客户端创建Socket实例，将进行TCP三次握手，握手完成后，实例对象才创建完成
	- 服务端创建ServerSocket实例，调用accept()方法，等待请求
- 数据传输
	- Socket实例都有一个InputStream和OutputStream，通过这两个对象交换数据

### 2.4 NIO的工作方式

- BIO带来的挑战（阻塞IO）
	- 一旦IO阻塞，线程将失去CPU的使用权
- NIO的工作机制 

（...............异步IO，重要，暂时未使用到）

### 2.5 I/O调优

- 磁盘I/O优化
	- 性能检测。Linux下iostat命令可查看wait指标,一般不应超过25%；还需要判断磁盘的IOPS能不能达到需求，IOPS=(磁盘数* 每块磁盘的IOPS)/（磁盘读的吞吐量+RAID因子* 磁盘写的吞吐量）
	- 提高I/O性能。增加缓存；优化磁盘管理系统（在底层操作系统层面考虑）；设计合理磁盘存储块及访问策略；合理利用RAID
- TCP网络参数调优
	- Linux中查看/proc/sys/net/ipv4/ip_local_port_range来获取可使用的端口范围。若可用端口偏少，可能导致大量请求等待建立连接
	- 若发现有大量TIME_WAIT连接，可调小/proc/sys/net/ipv4/tcp_fin_timeout的值来快速释放
	- cat /proc/net/netstat 查看TCP统计信息
	- cat /proc/net/snmp 查看系统的连接情况
	- netstat 查看网络统计信息

	![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/2-7.jpg)  
	![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/2-8.jpg)
- 网络I/O优化
	- 减少网络交互次数。设置缓存和合并访问请求
	- 减少网络传输数据量大小。数据压缩
	- 尽量减少编码。提前把字符转化为字节

	![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/2-9.jpg)

### 2.6 适配器模式
把一个类的接口变换成客户端所能接受的另一种接口，使两个不匹配接口的类能够在一起工作

- 适配器模式结构
	
	![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/2-10.jpg)
	- Target（目标接口）。所要转换的所期待的接口
	- Adaptee（源角色）。需要适配的接口
	- Adapter（适配器）
- Java I/O中的适配器模式
	- InputStreamReader和OutputStreamWriter为典型
	- 一般编程模式如下图

	![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/2-11.jpg)
	- [https://blog.csdn.net/zhangjg_blog/article/details/18735243](https://blog.csdn.net/zhangjg_blog/article/details/18735243 "适配器模式详解")

### 2.7 装饰器模式

[https://blog.csdn.net/zhaoyanjun6/article/details/56488020](https://blog.csdn.net/zhaoyanjun6/article/details/56488020 "装饰器模式详解")

装饰器模式和代理模式有点类似，都有转发的概念；代理模式是两个类实现同一个接口，一个类转发另一个类的方法；装饰器模式的前半部分类似代理模式，但主要在后面一系列的继承关系

### 2.8 适配器模式和装饰器模式区别

适配器和装饰器都叫做包装模式（Wrapper），都是包装一个类，但目的不一样。适配器是要将一个接口变成另外一个接口，一般都有一个新的接口；而装饰器则不改变原来的接口，只是增加原有对象的功能

# 3. 深入分析JavaWeb中的中文编码问题
### 3.1 常见的编码格式

- 计算机存储数据的单位是字节，可看作是英文字母。想要显示成中文或其它语言，就必须编码
- 编码格式
	- ASCII码
	- ISO-8859-1. 涵盖了大多数西欧语言字符，仍是单字节编码，总共256个字符
	- GB2312. 双字节编码，编码范围A1-F7，682个符号，6763个汉字
	- GBK. 拓展GB2312，编码和GB2312兼容，能表示21003个汉字
	- GB18030. 与GB2312兼容，应用不广泛
	- UTF-16. 双字节编码，16位所以叫UTF-16，不论什么字符都用两个字节表示（Java以UTF-16作为内存字符存储的格式，因为它简化了字符串操作）
	- UTF-8. UTF-16的缺点是可以用单字节表示的字符也要用双字节表示，浪费空间；所以UTF-8采用变长编码弥补这一缺点，不同字符的长度为1-6字节不等。编码规则如下

	![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/3-1.jpg)

### 3.2 Java中需要编码的场景

- I/O操作中存在编码
	- 一般都是用InputStreamReader或OutputStreamWriter做中间桥梁
- 内存操作中的编码
	- String和byte[]的转换，涉及到编码
	- byte[]和char[]的转换，涉及到编码，可用Charset类的encode和decode实现

### 3.3 Java中编解码
以下均以“I am 君山”为例说明编码细节
- ISO-8859-1编码

	![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/3-2.jpg)
	- 7个char字符经过ISO-8859-1编码后转换为7个byte，“君山”被转换成3f，即"?"字符，这会吞噬原来的中文字符信息无法复原，称为“黑洞”
- GB2312编码
	
	![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/3-3.jpg)	
	- 只支持有限的汉字，并不是所有汉字都能用GB2312编码
- GBK与GB2312兼容，一般的汉字编码结果相同
- UTF-16
	
	![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/3-4.jpg)
- UTF-8

	![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/3-5.jpg)
- 对中文字符，上述编码都能处理。GB2312与GBK类似，但GBK范围更大，尽量选择GBK；UTF-16编码效率比UTF-8高，但不适合在网络间传输，UTF-8效率较低但适合传输

### 3.4 在Java Web中的编解码

URL中的中文采用GBK编解码

（...........关于其它Web部分的编解码，暂时用不到）

### 3.5 JS中的编码问题

(..................暂时不用JS)

### 3.6 常见问题分析

- 中文变成看不懂的乱码
	- 字符串在解码时所用的字符集和编码时的字符集不一致

	![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/3-6.jpg)
- 一个汉字变成一个问号

	![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/3-7.jpg)
	- 中文经过了ISO-8859-1编码，超出范围的字符统一用3f表示，出现“黑洞”现象
- 一个汉字变成两个问号
	
	![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/3-8.jpg)
	- 多次混合解码
- 巧合情况
	
	![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/3-9.jpg)	

# 4. Javac编译原理

![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/4-1.jpg)

（..................编译原理部分，暂时不用）
（..................访问者模式，暂时不用）

# 5. 深入class文件结构

与《深入理解jvm》中第六章一样，可互补

# 6. 深入分析ClassLoader工作机制
除了将Class加载到JVM外，还能审查每个类应该由谁加载，还能将Class字节码重新解析成JVM统一要求的格式

### 6.1 ClassLoader类结构分析

- defineClass(). 将字节流解析成JVM能识别的Class对象
- findClass(). 直接覆盖父类的findClass()实现类的加载规则
- resolveClass()
- ClassLoader是抽象类，一般实现自己的加载器时都会继承URLClassLoader

### 6.2 ClassLoader的等级加载机制

上级委托接待机制：一个类由加载器加载，加载器都会先询问上一级加载器是否能加载，由此找到最高的一级能加载此类的加载器进行加载

JVM提供三层ClassLoader:
- Bootstrap ClassLoader. 主要加载JVM自身工作所需要的类，完全由JVM控制。没有父加载器也没有子加载器
- ExtClassLoader. 是JVM自身的一部分，但却有点不同，特定服务于java.ext.dirs下的类
- AppClassLoader. 父类是ExtClassLoader，加载一般的类，包括在java.class.path下的所有类。通过Launcher.getClassLoader()方法获取的就是AppClassLoader对象

自己实现的类加载器的父加载器都是AppClassLoader，所以等级层次如图![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/6-1.jpg)

JVM加载class文件到内存有两种方式：
- 隐式加载。 不通过在代码里调用ClassLoader来加载，而是JVM自动加载
- 显式加载。 显式调用ClassLoader,如getClassLoader().loadClass()或Class.forName()或自己实现的ClassLoader的findClass()等
- 一般两种方法是混合使用的，比如通过自定义ClassLoader显式加载一个类时，这个类引用了其他类，那其他类则是隐式加载

### 6.3 加载class文件
![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/6-2.jpg)

在被使用前，过程可分为加载、连接、初始化：
- 加载。 即加载字节码到内存，如何加载应在findClass()中实现
- 连接。 包括验证（确保格式、行为正确），准备（准备各字段、方法、接口所需的数据结构和内存），解析
- 初始化。 字段被初始化为默认值

### 6.4 加载类错误分析

- ClassNotFoundException. JVM在加载class文件到内存时，在classpath下找不到对应的文件。应确保每个类引用的类也都在classpath下
- NoClassDefFoundError. 如`java -cp example.jar Example`，原因是在指定类时没有加包名。应为`java -cp example.jar net.xxx.Example`
- UnsatisfiedLinkError. 通常是JVM启动时，不小心把JVM中的某个lib删除了
- ClassCastException. 强制类型转换时出现，表示不能这样转换
- ExceptionInInitializerError. 

### 6.5 常用的ClassLoader分析
（讲了Servlet中的StandardClassLoader和WebappClassLoader的加载过程..............暂时没用过）

主要思想其实就是双亲委派原则，找到可以加载的最高级加载器去加载

### 6.6 实现自己的ClassLoader

用自定义ClassLoader的情况：
- 在自定义路径下查找class文件
- 要对加载的类做特殊处理，如加密过的class文件，需要做解密来加载
- 定义类的实现机制（如类的热部署）

自定义ClassLoader一般代码结构（或直接继承URLClassLoader）：

	class MyClassLoader extends ClassLoader{
		...
		protected Class<?> findClass(String name) throws ClassNotFoundException{
			Class<?> aClass = findLoadedClass(name);//检查请求的类是否已经被加载
			if(aClass != null)
				return aClass;
			if(packageName.startsWith(name)){
				byte[] classData = 自定义获取class文件字节码(name);
				if(classData == null)
					throw new ClassNotFoundException();
				else
					return defineClass(...,classData,...);//字节码转成Class对象
			}else
				return super.loadClass(name);
		}
	}

### 6.7 实现类的热部署

JVM在加载类之前会检查类是否已被加载过，即调用findLoadedClass()进行检查。若已加载过，再调用loadClass()或defineClass()会导致冲突，发生异常。

JVM判断同一个类的两个条件：
- 全限类名一样（即包含包名的类名）
- 加载此类的ClassLoader是同一个实例

如：

	ClassLoader loader1 = new ClassLoader();
	Class r1 = loader1.findClass("a.class");
	ClassLoader loader2 = new ClassLoader();
	Class r2 = loader2.findClass("a.class");

假设ClassLoader类中没有检查类是否已被加载过，这不会引发异常，因为是两个不同的loader进行的加载，若改成：

	ClassLoader loader1 = new ClassLoader();
	Class r1 = loader1.findClass("a.class");
	Class r2 = loader1.findClass("a.class");

则会引发冲突异常（假设ClassLoader类中没有检查类是否已被加载过）

动态加载类的ClassLoader对象在没有被引用时会被回收，和普通对象一样；动态加载的类的字节码（即类本身）被放在JVM的PermGen区（即永久代，不是老年代，不是方法区）中，也会在FULL GC时被回收（需满足：它没有存活的实例；它的ClassLoader对象已被回收；没有反射引用它）

### 6.8 Java应不应该动态加载类

除6.6中说的三点理由外，不应该动态加载类。不可能把Java加载类动态到解释型语言那种地步（即只要修改下类，无需重启，程序运行就会发生变化）。在Java中，修改了一个类，就应该重启JVM来达到程序行为变化的目的

# 7. JVM体系结构与工作方式

只放两个主要结构图，其它和《深入理解jvm》中第八章一样，可互补

![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/7-1.jpg)

![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/7-2.jpg)

# 8. JVM内存管理

未记录的内容和《深入理解jvm》互补

### 8.2 内核空间和用户空间

计算的内存空间被分为内核空间和用户空间。内核空间提供给操作系统运行时所必要的内存需求，而程序只能使用用户空间的内存。所以如网络传输中，数据都是先从内核空间接收，然后再复制到用户空间

类本身（即类的class对象）和加载类的类加载器都放在永久代中（不是老年代）

### 8.7 内存问题分析

- jstat分析GC日志
- jmap+mat分析堆内存
- crash日志分析
	- EXCEPTION_ACCESS_VIOLATION。 JVM本身自己代码的BUG。大部分情况由垃圾回收导致
	- SIGSEGV。 本地代码或JNI的代码出错
	- EXCEPTION_STACK_OVERFLOW。 java代码和本地代码共用了相同的栈，然后出现了栈溢出

### 8.8~10 三个JVM分析实例

# 9. Servlet工作原理解析

### 9.1 从Servlet容器说起

如Jetty和Tomcat，都是Servlet容器。Tomcat容器模型如图：
	![](https://github.com/limbo-note/books/blob/master/JAVA_WEB/9-1.jpg)
真正管理Servlet的容器是Context容器，一个Context对应一个web工程。Tomcat配置中的`<Context path="..." docBase="..." reloadable="true" />`可以体现这一点

- Servlet容器启动过程
	- 