[TOC]

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