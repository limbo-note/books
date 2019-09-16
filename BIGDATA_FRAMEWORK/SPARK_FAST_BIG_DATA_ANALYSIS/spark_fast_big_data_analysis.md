[TOC]

# 3. RDD

- RDD基础
  - 转化操作（transformation）和行动操作（action）, Spark只有第一次在一个行动操作中用到时，才会真正计算。 
  - 默认情况下，每次的行动操作都会重新计算，想在多个行动操作中重用同一个 RDD，可以使用 RDD.persist() 让 Spark 把这个 RDD 缓存至内存或磁盘

  ![](3-1.jpg)

- 创建RDD
  - 读取外部数据集
  - 在驱动器程序中对一个集合进行并行化（把程序中一个已有的集合传给 SparkContext 的 parallelize() ） ，多用于开发测试

- RDD操作

  - 转化。返回一个全新的RDD，原RDD不会发生变化。转化出来的 RDD 是惰性求值的，只有在行动操作中用到这些 RDD 时才会被计算 
    - Spark用谱系图记录RDD之间的依赖关系，丢失RDD时依靠此谱系图来恢复

      ![](3-2.jpg)

  - 行动。大多数情况下， RDD 不能通过 collect() 行动操作收集到本地进程中，因为它们一般都很大
  
  - 惰性求值 
  
    - 不应该把 RDD 看作存放着特定数据的数据集， 而最好把每个 RDD 当作我们通过转化操作构建出来的、记录如何计算数据的指令列表 
    - 把数据读取到 RDD 的操作也同样是惰性的 ，如sc.textFile() 

![](3-3.jpg)

- 持久化

  ![](3-4.jpg)

# 4. 键值对操作

pairRDD，专有的RDD

- 创建pairRDD
  - mapToPair() ，parallelizePairs() 等
- 具体操作函数见书
  - 聚合操作。reduceByKey() ，foldByKey()  ，combineByKey() 
  - 分组操作。`rdd.reduceByKey(func)` 与 `rdd.groupByKey().mapValues(value => value.reduce(func))` 等价 ，但是前者更为高效，因为它避免了为每个键创建存放值的列表的步骤 
  - 连接操作。
  - 排序。
  - 行动操作基本与RDD一样

### 数据分区

假设一个不变的大表userData和每次都变化的小表events 做join操作。默认情况下，连接操作会将两个数据集中的所有键的哈希值都求出来， 将该哈希值相同的记录通过网络传到同一台机器上，然后在那台机器上对所有键相同的记录进行连接操作。

在每次调用userData.join(events )时，都会重新计算大表userData的哈希值进行混洗并发送到相应节点，这是很低效的。利用partitionBy() 预先将userData进行分区并持久化（不进行持久化的话，相当于每次依然需要重新混洗），可避免这种情况。

![](4-1.jpg)

![](4-2.jpg)

![](4-3.jpg)

- 自定义分区方式。实现Partitioner接口

