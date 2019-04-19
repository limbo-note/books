# 1. Redis #

- 键值对。值可以为字符串、哈希、列表、集合、位图等等
- 速度快。所有数据都放在内存中，C语言实现，单线程架构
- 持久化。RDB \ AOF写到硬盘
- 主从复制。

**Redis 基础配置**   
`logfile: 日志文件`  
`dir: Redis工作目录`  
`daemonize: 是否以守护进程方式启动`  

**命令**  
`redis-cli shutdown nosave\save`: 优雅关闭redis服务

# 2. API #
### 2.1 预备  
**全局命令**  
`keys *`：查看所有键，时间O(n)  
`dbsize`：键总数，时间O(1)  
`exists key`：1存在，0不存在  
`del key [key ...]`：删除键，无视数据结构类型，返回删除成功的键的个数  
`expire key seconds`：设置键的过期时间，到期自动删除  
`ttl key`：返回键的剩余过期时间，-1表示没有该键没有设置过过期时间，-2表示该键不存在  
`type key`：键的数据结构类型

**数据结构和内部编码**  
对外数据结构：string、hash、list、set、zset，图2-1    
内部编码：`object encoding key` 查询内部编码  
数据结构和内部编码对应关系，图2-2

**单线程架构**  
所有命令都进入一个队列，队列中的命令一条一条地被单线程执行
### 2.2 字符串  
**命令**  
`set key value [ex seconds] [px milliseconds] [nx|xx]`  
`mset key value [key value ...]`  
`mget key [key ...]`: 只有一次网络时间消耗  
计数操作：自增自减等  
`append, strlen, setrange, getrange`  

**内部编码**  
`int, embstr, raw`  

**使用场景**  
1. 缓存  
2. 计数  
3. 用户session管理  
4. 访问频率限制（每分钟不超过几次）  

### 2.3 哈希  
`hset, hget, hdel, hlen, hmget/hmset, hexists,  hkeys, hvals, hgetall, hincrby, hstrlen`  

**内部编码**  
`ziplist, hashtable`

### 2.4 列表
消息队列、文章列表

### 2.5 集合 
用户标签  

### 2.6 有序集合
排行榜系统

### 2.7 键管理
**单个键管理**  
键重命名：`rename, renamenx`  
随机返回一个键：`randomkey`  
键过期：`expireat, pexpire, pexpireat, pttl, persist`  
迁移键：`move, dump+restore, migrate`  

**遍历键**  
`keys/scan, hgetall/hscan, smembers/sscan, zrange/zscan` 

**数据库管理**  
`dbsize, select, flushdb/flushall`

# 3. 小功能大用处
### 3.1 慢查询分析
**发现执行时间太长的命令**  
发送命令->命令排队->命令执行->返回结果， 慢查询日志只统计命令执行的时间  
`slowlog-log-slower-than, slowlog-max-len`  
日志列表超过最大长度时，最早的日志将移出，插入最新的日志  
`slowlog get, slowlog len, slowlog reset`  

### 3.2 Redis Shell
**redis-cli**  
-r: repeat, 执行多次命令  
-i: interval, 每隔几秒执行一次命令  
`redis-cli -r 100 -i 1 info | grep used_memory_human`：看内存使用  
-x: 从标准输入读取数据作为redis-cli最后一个参数, `echo "world" | redis-cli -x set hello`  
-c, -a, --scan, --pattern  
--slave: 把当前客户端模拟成当前redis节点的从节点，可以记录更新操作  
--rdb, --pipe, --bigkeys(找到内存占用比较大的键), --eval  
--latency(-history/-dist): 测试客户端到目标redis的网络延迟  
--stat/info: 获取redis统计信息  
--raw/no-raw  

**redis-server**  
**redis-benchmark**  
可做性能测试  

### 3.3 Pipeline
1)发送命令->2)命令排队->3)命令执行->4)返回结果  
其中步骤1和步骤4加起来为RTT，pipeline通过将命令组装，可通过一次RTT完成操作  

**原生批量命令与pipeline区别**  

- 原生批量是原子的，pipeline非原子
- 原生批量是一个命令对应多个key， pipeline支持多个命令
- 原生批量是服务端实现的，pipeline是服务端客户端一起  

### 3.4 事务与Lua
**事务**  
要么全执行，要么全不执行。`multi, exec, discard, watch`  

**Lua**  
`eval, evalsha`

### 3.5 Bitmaps
`setbit, getbit, bitcount, bitop, bitpos`

### 3.6 HyperLogLog

### 3.7 发布订阅
`publish channel, subscribe channel, ...`

### 3.8 GEO

# 4. 客户端
### 4.1 客户端通信协议(RESP)
### 4.2 Jedis
- 直连, `new Jedis(...)`
- 连接池连接, `JedisPool.getResource()`  

**pipeline**  
1. `pipeline = jedis.pipelined();`  
2. `pipeline.get(...)`  
3. `pipeline.sync()` / `pipeline.syncAndReturnAll()`  这一步才真正执行命令  

**jedis使用lua**
### 4.3 Python的redis-py
### 4.4 客户端管理
`client list`：所有连接的客户端信息（表4-5）  

- 客户端标识
- 输入缓冲区大小（暂存要执行的命令）
- 输出缓冲区大小（暂存命令执行的结果）
- 客户端存活状态（已经连接的时长/最近一次空闲的时间）
- 客户端类型（普通/master/slave等）  

`info clients`：查看当前连接数等 
`client setName/getName, client kill, client pause`  
`monitor`：可以监测到所有执行的命令，但是可能会大量占用输出缓冲区 

**客户端相关配置**  
**客户端统计片段**  
`info clients, info stats`

### 4.5 客户端常见异常

1. 无法从连接池获取连接  
Jedispool默认最大连接8个，全部被占用时，连接就需要等待释放。若在规定时间内未等待到，则抛出异常。  资源全被占用的可能原因：
	- 最大连接数设置过小
	- 没有正确进行释放
	- 慢查询操作，归还速度慢
	- 服务端原因
2. 客户端读写超时(`Read timed out`)  
3. 客户端连接超时(`connect timed out`)  
4. 客户端缓冲区异常(`Unexpected end of stream`)  
	- 输出缓冲区满
	- 长时间闲置，被服务器主动断开
	- 不正常并发读写  
5. Lua脚本正在执行  
6. Redis正在加载持久化文件(`LOADING Redis is loading the dataset in memory`)  
7. Redis使用的内存超过设置  
8. 客户端连接数过大(`ERR max number of clients reached`)  

### 4.6 两个案例分析  
1. 主从复制主节点占用内存过大  
2. 慢查询导致的客户端连接超时  

# 5. 持久化
### 5.1 RDB  
`bgsave`：Redis进程执行fork创建子进程，负责完成持久化。默认采用LZF压缩算法。  

**自动触发bgsave：**  
1. save m n, m秒内n次修改  
2. 从节点全量复制，则主节点会bgsave并生成rdb发给从节点  
3. `debug reload`  
4. `shutdown`  

**流程**：  
父进程->fork->子进程->生成rdb->通知父进程  

**优缺点**：  
优：适用于备份，恢复时加载速度远快于AOF  
缺：无法实时持久化，频繁执行成本高  

### 5.2 AOF
以日志的方式将每次写命令记录下来，恢复时重新执行AOF文件中的命令达到恢复数据的目的，有实时性。  
默认不开启，开启需要配置`appendonly yes`.  

**流程**：  
命令写入(到缓冲区)->文件同步(到硬盘)->文件重写(定期,达到压缩的目的)->重启加载(较rdb,加载aof文件更优先)  

### 5.3 问题定位与优化  
1. fork耗时问题，需要复制父进程空间内存页表，与父进程内存占用成正比。  
2. 备份子进程开销监控和优化：  
redis属CPU密集型  
内存消耗分析P341： **如有多个实例，保证同一时刻只有一个子进程工作；避免在大量数据写入时做备份，会导致父进程维护大量页副本**   
硬盘开销分析  
3. AOF追加阻塞  

### 5.4 多实例部署  
利用`info Persistence`通过外部程序轮询控制持久化操作，保证每个实例持久化串行执行  

# 6. 复制
### 6.1 配置
1. 配置文件 slaveof [host] [port]  
2. redis-server --slaveof [host] [port]  
3. 运行期 动态 slaveof [host] [port]   
均为异步命令, `info replicaiton`查看复制相关状态  

`slaveof no one`断开复制关系，将从节点晋升为主节点。不会抛弃原有数据，只是不再复制数据。  
`slaveof`还可用于切主，流程：断开原主->与新主建立关系->删除原主数据->复制新主数据  
传输延迟：同机房，关闭`repl-disable-tcp-nodelay`；跨机房，则打开。  

### 6.2 拓扑  
一主一从、一主多从、树状主从  

### 6.3 原理  
**复制过程**：保存主节点信息(异步，保存后直接返回)->主从建立socket连接->发送ping命令(等待pong回复)->权限验证->同步数据集->命令持续复制  

**数据同步(psync)** ：全量复制，部分复制  
1. 复制偏移量：判断主从节点数据是否一致，差值可判定当前复制的健康度  
2. 复制积压缓冲区(定长队列)   
3. 主节点运行ID(Redis节点标识，`info server`查看)  
4. psync命令：`psync [runid] [offset]`  全量复制就是psync [runid] -1  

**全量复制最主要的耗时步骤**： 主节点bgsave，rdb网络传输，从节点清空原数据，从节点加载rdb，从节点可能的AOF重写  

**心跳**： 主从节点建议复制后，维护着长连接并发送心跳命令(ping命令、replconf ack命令)  

**异步复制**  

### 6.4 开发与运维中的问题  

**读写分离**(减轻主节点压力，只对主节点执行写操作)：复制数据延迟、读到过期数据、从节点故障  
**主从配置不一致**：maxmemory、hash-max-ziplist-entries  
**规避全量复制**  
**规避复制风暴**  

# 7. Redis的噩梦：阻塞
### 7.1 发现阻塞  
客户端抛出超时异常——JedisConnectionException  
redis监控系统——CacheCloud：命令耗时、慢查询、持久化阻塞、连接拒绝、CPU/内存/网络/磁盘  
**内在原因**  
API、数据结构使用不合理：慢查询(如hgetall)——slowlog get [n]查看  
CPU饱和：redis-cli -h -p --stat查看redis状态(可看到OPS)，过度追求内存优化从而导致了CPU消耗大，bgsave持久化占用大量CPU  
持久化阻塞：fork阻塞、AOF刷盘阻塞、HugePage写操作阻塞  

**外在原因**  
CPU竞争：进程竞争(bgsave竞争，其他进程竞争)，绑定CPU(为减小多实例redisCPU切换开销做绑定，在做持久化时父子进程共享CPU，造成阻塞)  
内存交换：`cat /proc/[process_id]/smaps |grep Swap`查看进程内存交换  
网络问题：连接拒绝、网络延迟(--latency可做测试)、网卡软中断

# 8. 理解内存
### 8.1 内存消耗
`info memory`获取内存相关指标，关注used_memory_rss和used_memory  
1. 自身内存：一般自身内存消耗非常小  
2. 对象内存：内存占用最大的一块，存储所有数据  
3. 缓冲内存：客户端缓冲（输入输出）、复制积压缓冲区、AOF缓冲区（重写期间的写入命令）  
4. 内存碎片：频繁做更新操作、大量过期键删除——解决：数据对齐、安全重启  
5. 子进程内存消耗：**当父进程处理写请求时会对需要修改的页复制出一份副本完成写操作，而子进程依然读取fork时整个父进程的内存快照（THP优化，建议关闭）**  

### 8.2 内存管理
1. 设置内存上限(maxmemory)  
2. 动态调整内存上限  
3. 内存溢出控制策咯（noeviction、volatile-lru、allkeys-lru、allkeys-random、volatile-random、volatile-ttl）  

### 8.3 内存优化
redisObject优化、缩减键值对象、共享对象池、字符串优化、编码优化、控制键的数量

# 9. 哨兵
和主从复制模式相比，只是多了一些sentinel监控节点，对主从复制数据节点进行监控，完成自动故障转移的功能，形成了Redis Sentinel架构  

# 10. 集群
(............)

# 13. 监控平台CacheCloud
### 13.1 部署
(自己构建)maven3, redis3.0, mysql 5, jdk7, cachecloud  
1. mysql创建cache_cloud库, 表结构cachecloud/script/cachecloud.sql  
2. cachecloud-open-web/src/main/swap目录下，配置  
3. mvn clean compile install [-Ponline][-Plocal] 构建，再启动  
4. 登录界面  

(直接使用二进制版本)  
- cachecloud-open-web-1.0-SNAPSHOT.war: cachecloud war包
- cachecloud.sql: 数据库schema，默认数据名为cache_cloud，可以自行修改
- jdbc.properties：jdbc数据库配置，自行配置
- start.sh：启动脚本
- stop.sh： 停止脚本
- logs：存放日志的目录  
默认端口是8585，可以修改start.sh中的server.port进行重置
