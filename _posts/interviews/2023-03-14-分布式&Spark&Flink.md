---
title: 分布式&Spark&Flink面试题
author: Yu Mengchi
categories:
  - Interview 
  - 面试知识点
  - 系统设计
tags:
  - Interview
  - 分布式系统
---
  
### 有哪些分布式一致算法？分布式一致性协议？

分布式一致性算法：是在分布式系统中保持一致性的技术，确保在分布式环境中即使出现故障，比如网络分区、节点崩溃等，系统仍然能够保持一致性或最终一致性。

1. Paxos
- 核心思想：
  - 通过多轮提案(proposal)机制，在多个节点之间达成一致。
  - 包括三个角色：Proposer、acceptor、Learner。
2. Raft
- 核心思想：
  - 将共识问题分解为领导选举、日志复制、安全性三个子问题。
  - 集群中有一个明确的Leader节点负责处理客户端请求，并将日志同步到其他follower节点。
- 应用场景：Etcd，Consul
3. ZAB
- 核心思想：
  - 类似于Raft，也采用Leader-follower模型，leader负责广播事务日志
  - 强调顺序性和原子性，确保所有节点按照相同的顺序执行事务
- 应用场景：ZooKeeper，用于分布式锁、配置管理、服务发现。
4. 2PC
- 核心思想：
  - **准备**阶段：协调者询问所有参与者是否可以提交事务
  - **提交**阶段：如果所有参与者都同意，则协调者通知提交；否则回滚
- 特点：存在单点故障
- 应用场景：MySQL的XA事务
5. 3PC
6. CRDT：用于共享白板、协同文档
7. BFT：拜占庭容错算法，区块链
8. Chord：将数据分布到一个逻辑环上，每个节点负责一部分数据，P2P文件共享系统

---
### 系统的CAP性质？如何保证AP和CP？

在分布式系统设计中，CAP理论指出任何网络分区下的分布式系统最多只能满足以下三个条件中的两个：

- Consistency：一致性，所有节点在同一时刻看到的数据都是相同的；
- Availability：可用性，系统是可用的，每一个请求都能接收到响应；
- Partition Tolerance：分区容忍性，即使发生了网络分区，即部分节点之间无法通信，系统仍然能够继续运行。

如何保证AP：保证AP意味着需要接收一定程度上的数据不一致。最终一致性模型
- 异步复制：主节点处理写请求，并同步到副本节点。
如何保证CP：牺牲一定的可用性，发生网络分区时，某些服务暂时不可用直到分区解决。
- 同步复制：所有写操作必须被所有副本确认后才算成功完成，保证强一致性
- Paxos/Raft算法
- 两阶段提交2PC

> 对于一致性要求极高的交易性数据采用CP模式；
> 对于用户体验影响较大但实时性要求较高的查询采用AP模式；

---
### Paxos算法原理讲解一下？

Paxos将节点分成三个角色：Proposer、Acceptor、Learner
- Proposer提出提案，每个提案包含一个唯一的编号和要达成一致的值；
- Acceptor接收或拒绝提案；
- Learner学习已经达成一致的结果，不参与决策过程，只接收最终的结果。

算法流程：
1. Prepare阶段

2. Accept阶段

---
### ZooKeeper等应用分布式协议的中间件有哪些？
- ZooKeeper：使用ZAB协议
- Etcd：使用Raft一致性算法

---
### 使用Spark有什么需要注意的地方？

1. 内存管理：Spark依赖内存，配置集群的内存设置是很重要的
- 配置合理的Executor和Driver内存大小(通过spark.executor.memory和spark.driver.memory)
- 尽量减少Shufffle操作：控制Shuffle操作的数量，因为Shuffle会消耗大量内存
- 选择合适的序列化：Spark使用序列化在集群节点间传输数据，使用Kryo序列化(spark.serializer=org.apache.spark.serializer.KryoSerializer)以减少序列化和反序列化的开销
2. 数据分区：Spark将数据分区到集群的不同的节点上，并行处理，理解数据是怎么分区的，怎么控制分区实现最优性能表现
- 确保分区数量合理。分区过多会导致过多的任务调度开销；分区过少会降低并行度；
- 使用repartition或coalesce调增分区数，但要注意repartition会触发Shuffle，而coalesce不会
- 避免数据倾斜，即某些分区的数量远大于其他分区。可以通过自定义分区器或预处理数据来缓解。
3. Shuffle操作
- 尽量减少Shuffle操作(避免不必要的groupByKey和join)
- 使用reduceByKey替代groupByKey，因为reduceByKey会在Map端进行部分聚合
- 调整Shuffle参数，例如spark.shuffle.partitions默认值为200，可以根据数据规模调整
4. 使用高效的数据存储格式：比如Parquet、Avro、ORC，这些支持列式存储和压缩，使用HDFS
5. 调整合适的并行度：每个CPU核心对应2~4个任务，调整spark.default.parallelism和spark.sql.shuffle.partitions优化并行度
6. 利用广播变量来优化小表join大表的场景。 
7. 缓存：Spark会缓存数据到内存，理解缓存是怎么工作的，并且什么时候需要使用缓存
8. 资源分配：使用YARN管理资源。Spark允许为特定任务分配资源。理解怎么分配资源

---
### Spark广播怎么使用？Spark需要join一个大表和一个小表时应该怎么优化使用？

可以使用广播变量来优化性能。广播小表可以避免将其复制到每个工作节点上，从而减少了通信开销和内存使用。

1. 将小表广播到集群中所有的节点

```scala
small_table = sc.parallelize([(1, "a"), (2, "b"), (3, "c")])
broadcast_table = sc.broadcast(small_table.collectAsMap())
```

2. 在大表上执行map操作，将小表中的数据与大表中的数据进行join

```scala
large_table = sc.parallelize([(1, "x"), (2, "y"), (3, "z"), (4, "w"), (5, "v")])
result = large_table.map(lambda x: (x[0], (x[1], broadcast_table.value.get(x[0])))).collect()
```

> 在join操作时，也可以手动关播小表，显示调用broadcast(smallDF)函数；spark.sql.autoBroadcastJoinThreshold广播阈值参数设置小表大小阈值；如果小表还是很大，但还是想广播，可以压缩小表；

---
### Spark遇到数据倾斜怎么处理？

数据倾斜：某些分区的数据大于其他分区

(1) 先确认是否存在数据倾斜

- 先通过Spark UI查看任务执行时间分布，如果某个任务的执行时间远大于其他任务，那么可能有数据倾斜；
- 使用日志或调试工具查看每个分区的数据量；
- 在Shuffle操作后查看每个分区的数据量；
```scala
rdd.mapPartitions(iter => Array(iter.size).iterator).collect().foreach(println)
```

(2) 解决数据倾斜方法

1. 调整分区数：
- 通过repartition或coalesce方法增加分区数量，把数据分散到更多的分区里
- 调整默认分区数：修改spark.sql.shuffle.partitions参数调整默认分区大小
2. 如果数据集中存在一些热点数据，可以在key上加随机前缀、哈希方法来对key加盐，分散这些key的数据分布，后续操作再删除随机前缀，恢复原始数据。从而避免热点数据集中在同一个分区中的情况。
3. 自定义分区器：通过继承Partitioner类重写getPartition(key: Any):Int方法实现自定义的分区逻辑，更灵活控制数据分布
4. 过滤异常数据：如果数据倾斜是因为有异常数据，比如超大key或脏数据，可以直接过滤这些数据
7. 广播变量：当进行大表与小表的join操作时，可以广播小表数据，避免Shuffle操作。

---
### 使用随机前缀解决步骤
1. 将数据集按照某个关键字进行分组，例如按照用户id进行分组
2. 对每个分组的关键字进行随机前缀处理，例如在用户id前添加一个随机字符串
3. 将处理后的数据集进行重新分区，使每个分区中的数据量均匀分布
4. 对每个分区中的数据进行聚合操作，例如求和或计数
5. 将每个分区的结果进行汇总，得到最终的结果

例子：
```scala
import org.apache.spark.sql.SparkSession
import scala.util.Random

object DataSkewHandling {
  def main(args: Array[String]): Unit = {
    // 初始化 SparkSession
    val spark = SparkSession.builder()
      .appName("Data Skew Handling with Random Prefix")
      .master("local[*]")
      .getOrCreate()

    import spark.implicits._

    // 创建一个示例 RDD，模拟数据倾斜
    val data = Seq(
      ("A", 1), ("A", 2), ("A", 3), ("A", 4), ("A", 5),
      ("B", 1), ("B", 2),
      ("C", 1)
    )
    val rdd = spark.sparkContext.parallelize(data)

    // Step 1: 为 Key 添加随机前缀（加盐）
    val saltedRdd = rdd.map { case (key, value) =>
      val salt = Random.nextInt(10) // 随机生成 0~9 的盐
      (s"${salt}_$key", value)     // 将盐与原 Key 拼接
    }

    // Step 2: 执行 shuffle 操作（例如 reduceByKey）
    val aggregatedRdd = saltedRdd.reduceByKey(_ + _)

    // Step 3: 去掉随机前缀，恢复原始 Key
    val resultRdd = aggregatedRdd.map { case (saltedKey, value) =>
      val originalKey = saltedKey.split("_")(1) // 提取原始 Key
      (originalKey, value)
    }

    // Step 4: 再次聚合以得到最终结果
    val finalResultRdd = resultRdd.reduceByKey(_ + _)

    // 输出结果
    finalResultRdd.collect().foreach(println)

    // 关闭 SparkSession
    spark.stop()
  }
}
```

> 注意如果原始key中就有"_"符号，那么在恢复原始key时会出现错误，这个时候可以(1)使用key中不存在的字符拼接随机前缀和key；(2)使用截取固定长度获取原始key，比如随机前缀是1长度，则获取1后面的字符串为原始key；

---
### Spark的架构和组件？

1. Driver：主程序，负责作业的提交和调度；

2. Cluster Manager：管理集群资源

3. Worker Node：集群中的worker节点，运行executor进程

4. Executor：负责执行任务并管理资源

5. RDD、DataSet、DataFrame：
- RDD：表示一个分布式数据集合，支持两种操作，Transformation(map,filter,flatMap)、Action(collect,count,saveAsTextFile)，不可变，但是可以通过转换操作生成新的RDD；弹性的，支持自动容错和数据恢复；分布式的，数据分布在多个节点上；可以从HDFS数据源加载数据；
- DataFrame：类似于数据库的表，支持SQL操作；高效查询；支持SQL查询；支持DataFrame操作，select,filter,join,groupBy
- DataSet：类型安全的数据集合，结合了RDD和DataFrame的优点，适合需要类型安全和高性能优化的场景。

**Spark任务执行流程**：
1. Driver

**Job，Stage，Task区别**：
1. Job是一个完整的计算任务，包含多个Stage
2. Stage包含多个Task，表示一个阶段
3. Task是运行在Executor上的实际最小执行单元

---
### Spark的基本数据结构？RDD、DataFrame和DataSet

- RDD：表示一个分布式数据集合，支持两种操作，Transformation(map,filter,flatMap)、Action(collect,count,saveAsTextFile)，不可变，但是可以通过转换操作生成新的RDD；弹性的，支持自动容错和数据恢复；分布式的，数据分布在多个节点上；可以从HDFS数据源加载数据；
- DataFrame：类似于数据库的表，支持SQL操作；高效查询；支持SQL查询；支持DataFrame操作，select,filter,join,groupBy
- DataSet：类型安全的数据集合，结合了RDD和DataFrame的优点，适合需要类型安全和高性能优化的场景。

---
### Spark SQL

---
### Spark的优化和调优？数据倾斜、并行度、数据压缩、缓存
executor memory
number of executors
Spark有很多可以调优的配置参数，以下是一些常用的参数：
1. spark.driver.memory：设置Driver进程的内存大小，可以根据具体应用场景进行调整。
2. spark.executor.memory：设置每个Executor进程的内存大小，可以根据机器配置和任务需求进行调整。
3. spark.executor.instances：设置Executor进程的数量，可以根据集群的资源和任务的并行度需求进行调整。
4. spark.serializer：设置序列化器，可以选择Kryo或Java序列化器，根据具体应用场景进行选择。
5. spark.shuffle.memoryFraction：设置Shuffle操作使用的内存比例，默认为0.2，可以根据具体应用场景进行调整。
6. spark.default.parallelism：设置默认并行度，即RDD的分区数，默认为CPU核心数，可以根据集群规模和任务复杂度进行调整。
7. spark.sql.shuffle.partitions：设置SQL操作的Shuffle的分区数，默认为200，可以根据数据大小和集群规模进行调整。
8. spark.storage.blockManagerHeartBeatMs：设置BlockManager的心跳间隔时间，可以根据集群的网络延迟和任务的数据交互量进行调整。
9. spark.streaming.blockInterval：设置Streaming数据块的时间间隔，默认为200ms，可以根据具体应用场景和数据流的特点进行调整。
10. spark.streaming.kafka.maxRatePerPartition：设置每个分区每秒读取的最大数据量，默认为无限制，可以根据数据量和集群规模进行调整。

以上是Spark中常用的一些调优参数，可以根据具体场景进行调整，以达到最优的性能和效果。

---
### Spark的部署和集群管理？

---
### Spark的实战应用

---
### Flink有哪些优化的措施
以下是一些Flink可以进行的优化措施：

1. 并行度调整：调整任务的并行度，以提高执行效率和吞吐量。
2. 内存管理：使用Flink的堆外内存管理机制来减少垃圾回收的开销，提高处理性能。
3. 数据倾斜解决方案：处理数据倾斜的方法包括打散、打分区、局部聚合、二次分区等，可以根据具体情况选择合适的方法来解决数据倾斜问题。
4. 合理使用缓存：在某些场景下，可以使用缓存来减少网络传输和IO开销，提高处理效率。
5. 文件系统选择：选择合适的文件系统来存储数据，以保证数据的可靠性和读写性能。
6. 状态管理：使用状态后端来管理Flink应用程序中的状态，以实现快速恢复和高可用性。
7. 合理配置检查点：检查点机制可以保证数据的一致性和可恢复性，合理配置检查点的间隔和超时时间，可以提高应用程序的性能和可靠性。
8. 使用数据结构：Flink支持多种数据结构，如Broadcast State、List State、Map State等，可以根据具体应用场景选择合适的数据结构，以提高处理效率和减少内存开销。
9. 流水线优化：对于复杂的任务流程，可以采用流水线优化技术，将多个操作合并为一个流水线，减少数据在内存中的复制和传输，提高处理效率。
   通过执行以上措施，可以显著提高Flink应用程序的性能和可靠性。

---
### Flink任务通常会出现哪些问题？

在Flink任务中，常见的问题包括：
1. 数据倾斜：当输入数据不平衡时，会导致某些任务的负载过重，导致处理速度变慢，甚至整个任务崩溃。
2. 内存溢出：当Flink任务处理大量数据时，可能会导致内存溢出，需要合理配置内存参数，使用堆外内存等机制来减少内存开销。
3. 网络延迟：当任务中涉及到网络传输时，可能会出现网络延迟问题，需要优化网络传输机制，减少网络传输开销。
4. 数据丢失：当任务中出现数据丢失时，需要检查任务的容错机制是否正确配置，以及数据源是否正确。
5. 任务调度问题：当任务调度不合理时，可能会导致任务不均衡，需要优化任务调度机制，以提高任务执行效率。
6. 状态管理问题：当任务中涉及到状态管理时，需要合理配置状态后端，以保证状态的可靠性和恢复性。
7. 并发控制问题：当任务中涉及到并发控制时，需要合理配置并发控制机制，以避免出现竞争条件和死锁等问题。
8. 应用程序设计问题：当应用程序设计不合理时，可能会导致性能下降和代码复杂度增加，需要合理设计应用程序架构，以提高应用程序的可维护性和可扩展性。
   针对以上问题，需要根据具体情况采取相应的优化措施，以保证Flink任务的稳定性和高效性。

---
### 数据仓库和数据湖的区别？
数据湖主要存放的是非结构化或半结构化数据。例如log文件、社交媒体数据、传感器数据等。

---
### 什么是ocpx广告？
Yes, I am familiar with OCPX advertising. 
OCPX stands for "Optimized Cost Per Click," which is a type of advertising model that uses machine learning algorithms to optimize the cost per click of an ad campaign. 
With OCPX, the advertiser sets a target cost per click, and the algorithm automatically adjusts the bid for each ad placement based on the likelihood of a click and the value of that click to the advertiser. 
This helps to maximize the return on investment (ROI) of the ad campaign and improve the overall performance of the advertising strategy.
