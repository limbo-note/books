[TOC]

# 4. 主题与分区

### 4.1 主题管理

#### 创建

`auto.create.topics.enable`  为true时，根据`num.partitions`  和`default.replication.factor`  来自动创建主题和分区

![](4-1.jpg)

可查看各机器上的日志文件或者直接看zk的`/brokers/topics/`  目录获取分区分配情况

zk的`/config/topics/`存储topic的配置

`--replica-assignment`  指定分区的副本分配到哪些broker

源码KafkaConfig.scala中可看到所有默认配置

Kafka创建主题会自动将名称中的`.`转化成`_`

kafka-topics.sh 脚本的实质就是调用`kafka.admin.TopicCommand` 类，然后在zk上相应的目录写入相关信息（**仅此而已，真正创建的实质性动作是由控制器读取zk来异步完成的，所以可以直接在zk中写入信息来完成创建**），也可以在代码中引入TopicCommand类进行主题操作，但更优的方式是使用KafkaAdminClient类。

#### 分区副本的分配

- 未配置机架信息`broker.rack`  或禁用机架`disable-rack-aware`时，采用的是未指定机架的分配策略，源码在`kafka.admin.AdminUtils.assignReplicasToBrokersRackUnaware ()`

    ![](4-2.jpg)

- 配置了机架信息的分配，会优先将分区的不同副本分配到不同的机架上：

    ![](4-3.jpg)

对于基于 key 计算的主题而言，建议在一开始就设置好分区数量，避免以后对其进行调整  

配置相关命令见书

#### 主题端参数

与主题相关的所有配置参数在 broker 层面都有对应参数，如果没有修改过主题的任何配置参数，那么就会使用 broker 端的对应参数作为其默认值。可以在创建主题时覆盖相应参数的默认值，也可以在创建完主题之后变更相应参数的值  

必须将 `delete.topic.enable`参数配置为 true 才能够删除主题 ，不能删除内部主题和不存在的主题

使用 `kafka-topics.sh` 脚本删除主题的行为本质上只是在 ZooKeeper 中的 `/admin/delete_topics` 路径下创建一个与待删除主题同名的节点，以此标记该主题为待删除的状态。与创建主题相同的是，真正删除主题的动作也是由 Kafka 的控制器负责完成的

![](4-4.jpg)

### 4.2 KafkaAdminClient

![](4-5.jpg)

![](4-6.jpg)

主题合法性验证保证主题创建（代码方式）的规范性，`org.apache.kafka.server.policy.CreateTopicPolicy` 接口 

### 4.3 分区管理

