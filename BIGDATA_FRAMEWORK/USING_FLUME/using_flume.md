[TOC]

# 2. Flume处理流数据

### Flume存在的意义

若将生产者直接与Hadoop等系统接应写入数据，可能出现丢失数据、数据处理速度不匹配的问题

可拓展性强，通过简单地增加Agent就能拓展服务器的数量

### Flume Agent内部原理

![](2-1.jpg)

![](2-2.jpg)

### Flume Agent配置

见书

### 复杂的流

![](2-3.jpg)

### 复制数据到不同的目的地

![](2-4.jpg)

### 动态路由

![](2-5.jpg)

### 关于事务

见书，看不明白。大概是每次写入或读取channel数据，都会由一个事务来封装，保证数据要么全部写入要么回滚？

### Agent失败和数据丢失

![](2-6.jpg)

当某一层的数据传输失败时，数据将被缓冲直至Channel被填满。此时开始回退到前一层，直到所有层都被填满。此时的数据将会发生丢失

### 批量和重复

同kafka等，为减少额外的网络传输或系统调用的开销比例，一般会将多个事件一起组成一个批次再进行传输（存储）。类似redis的pipeline

在sink端，若RPC调用在超时时间内没有得到响应且实际的RPC并没有失败，就会引发重试，进而重复

# 3. Source

### Source生命周期

![](3-1.jpg)

### 各种Source

- Sink-to-Source通信
  
  - 可以使用sink组和sink处理器，来发送数据给多个其他的Agent
  
- Avro Source
  - flume主要的RPC source
  - 配置示例见书
  
- Thrift Source
  
  - 跨语言的RPC source，解决Avro Source只支持java的问题
  
- RPC Source失败处理

- HTTP Source

  - 重写HTTPSourceHandler接口，来自定义handler处理程序

    ![](3-2.jpg)

  - 将HttpServletRequest的数据处理，转换成一组Event事件返回

  - handler处理类可以在configure()方法中配置参数

  - 处理程序抛出的一些异常可以自动对应到一些HTTP状态码并返回给客户端（类似Springmvc）

  - 默认的处理程序处理特定格式的JSON数据

  - 可参考学习HTTPSourceXMLHandler类源码

- Spooling Directory Source
  - 监听目录中的文件
- 使用Deserializers读取自定义格式
  - 反序列化器转化目录中文件的数据

- Syslog Source
- Exec Source
  - 执行用户配置的命令，基于命令的标准输出来生成事件
- JMS Source
  - 获取Java消息服务队列的数据，如ActiveMQ，Kafka
  - 转换JMS消息为Flume事件
- 自定义的Source
  - event-driven 和 pollable 两种类型的自定义source

# 4. Channel

