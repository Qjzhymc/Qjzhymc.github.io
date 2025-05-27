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
### spark运行机制？怎么读取文件？


---
### 使用Spark有什么需要注意的地方？

1. 内存管理：Spark依赖内存，配置集群的内存设置是很重要的
- 配置合理的Executor和Driver内存大小(通过spark.executor.memory和spark.driver.memory)
- 尽量减少Shufffle操作：控制Shuffle操作的数量，因为Shuffle会消耗大量内存
- 选择合适的序列化：Spark使用序列化在集群节点间传输数据，使用Kryo序列化(spark.serializer=org.apache.spark.serializer.KryoSerializer)减少序列化和反序列化的开销
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

### reduceByKey与groupByKey有什么区别？
1. reduceByKey包含了分组和聚合两个操作，groupByKey只是分组；所以一般groupByKey之后还会接一个map操作，对分组后的每个组进行聚合。
2. 而且reduceByKey在shuffle之前对分区内相同的key的数据集会先进行预聚合，性能会比较高。

```scala
val words = Array("one", "two", "two", "three", "three", "three")
val wordPairsRDD = sc.parallelize(words).map(word => (word, 1))

val wordCountsWithReduce = wordPairsRDD
  .reduceByKey(_ + _)
  .collect()

val wordCountsWithGroup = wordPairsRDD
  .groupByKey()
  .map(t => (t._1, t._2.sum))
  .collect()
```

---
### spark RDD
RDD是Spark的弹性分布式数据集，表示一个元素集合，RDD里的数据可以分布在不同的节点上。

核心特性：
1. 不可变：对RDD做转换之后会生成新的RDD；
2. RDD的数据是分布在不同分区的，每个分区分布在集群不同的节点上；
3. RDD有自动的容错机制，假设某个分区数据丢失，还是可以重新计算，重建出来；
4. RDD支持很多操作，比如map、filter、union、groupByKey、reduceByKey

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

> 在join操作时，也可以手动广播小表，显示调用broadcast(smallDF)函数；spark.sql.autoBroadcastJoinThreshold广播阈值参数设置小表大小阈值；如果小表还是很大，但还是想广播，可以压缩小表；

---
### Spark中大表join小表怎么操作

可以把小表数据变成广播变量，这样每个executor都有小表的数据，然后再大表数据处理的时候，根据大表数据的key从小表广播变量里找对应的数据。

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
### 数据倾斜？
1.现象：绝大多数task执行很快，某个task执行很慢
2.发生原理：在进行shuffle的时候，必须将相同的key拉到同一个节点上处理，比如按key聚合或join操作，此时如果某个数据量很大的话，就会发生数据倾斜。
3.定位导致倾斜的代码：可以再yarn-cluster上的spark web ui上查看task执行情况，耗时，数据量等。
一般只要看到Spark代码中出现了一个shuffle类算子（比如group by语句），那么就可以判定，以那个地方为界限划分出了前后两个stage。
4.查看key的分布情况：可以在Spark作业中加入查看key分布的代码，比如RDD.countByKey()。
5.采用随机前缀：将这个key对应的每条数据加随机前缀，将分散后的key分散到不同节点上，再join一起。


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

> 注意如果原始key中就有"_"符号，那么在恢复原始key时会出现错误，这个时候可以
 
> (1)使用key中不存在的字符拼接随机前缀和key；

> (2)使用截取固定长度获取原始key，比如随机前缀是1长度，则获取1后面的字符串为原始key；

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
### 作业优化问题？

---
### 有没有遇到spark任务调优问题？
**会有哪些问题：**
- 某个节点OOM了；
- 执行时间很慢；

**问题如何解决：**
- 查看task执行的日志，耗时、数据量。
- 然后判断是发生了什么问题，是数据倾斜、还是网络问题、还是外部IO阻塞的问题。

---
### Spark的部署和集群管理？

---
### Spark的实战应用

---
### Spark工作流程

---
### Spark shuffle是什么？哪些操作会触发shuffle？为什么shuffle对性能影响巨大？
本质是数据重分布过程，需要讲分散在集群不同节点上的数据按照特定规则进行重组时，就需要shuffle

会触发shuffle的操作：
1. repartition
2. 聚合操作：groupByKey,reduceByKey,aggregateByKey
3. 关联操作：join,cogroup
4. 去重和排序：distinct,sortByKey

以下操作不会触发shuffle：
1. 转换操作：map,filter,flatMap
2. 就地压缩：coalesce(shuffle=false)
3. 合并操作：union

shuffle影响性能的核心原因
1. 磁盘IO：中间结果需写入磁盘
2. 网络传输：数据需跨节点传输
3. 序列化开销：数据序列化和反序列化
4. 内存压力：聚合操作需大量内存

---
### 为什么有shuffle

**什么是shuffle：**
shuffle就是把多个节点上的同一个key，拉取到同一个节点上，进行聚合或join操作，比如reduceByKey，join操作，都会触发shuffle操作。

**为什么会有shuffle：**
因为RDD是分布式数据集，数据分布在多个节点上，如果要执行聚合就得把其他节点上的数据通过网络拉取到一起进行操作。


---
### 描述Spark Shuffle的演变历程，并比较不同shuffle实现的优缺点
1. 第一代：Hash-based Shuffle
 - 工作原理：每个Map任务为每个Reduce任务创建一个文件
 - 文件数量：M*R个文件
 - 优点：实现简单，不需要排序
 - 缺点：产生大量小文件，导致严重的文件系统压力和内存开销
2. 第二代：Sort-based Shuffle
 - 工作原理：每个Map任务只生成一个数据文件和一个索引文件
 - 文件数量：2*M个文件
 - 优点：减少文件数量，缓解文件系统压力
 - 缺点：额外的排序开销，内存使用增加
3. 第三代：Tungsten-Sort Shuffle
 - 工作原理：利用堆外内存和二进制格式处理数据
 - 优点：内存使用更高效，GC压力减小，排序性能更好
 - 缺点：实现复杂，对序列化有特定要求

---
### 描述Shuffle的write和read过程
Shuffle Write流程：
1. Map任务执行：首先执行Map阶段的计算逻辑
2. 内存缓冲：结果写入内存缓冲区(由spark.shuffle.file.buffer控制，默认32KB)
3. 排序分区：在Sort-based Shuffle中，对缓冲数据按partition ID进行排序
4. 溢出写入：当内存缓冲区达到阈值时，溢出到磁盘
5. 合并文件：Map任务完成时，所有溢出文件合并为一个文件
6. 生成索引：创建索引文件，记录每个分区的偏移量

Shuffle Read流程：
1. 获取数据位置：Reduce任务从MapOutputTracker获取数据位置信息；
2. 拉取数据：通过BlockManager和TransferService拉取远程数据
3. 聚合处理
4. 外部排序：如果数据过大无法放入内存，使用外部排序
5. 结果计算：完成Reduce逻辑计算并输出结果

---
### Spark Shuffle的关键配置参数有哪些

1. spark.shuffle.file.buffer:Map端写缓冲区大小
2. spark.reducer.maxSizeInFlight:Reduce端单次请求最大数据量
3. spark.shuffle.io.maxRetries:Shuffle读取失败最大重试次数
4. spark.shuffle.io.retryWait:重试等待时间
5. spark.shuffle.compress:是否压缩Shuffle输出
6. spark.shuffle.spill.compress:是否压缩溢出文件
7. spark.shuffle.service.enabled:是否启用外部shuffle服务
8. spark.sql.shuffle.partitions:SQL操作的shuffle分区数
9. spark.shuffle.sort.bypassMergeThreshold:绕过排序的分区阈值

---
### 如何识别Spark Shuffle过程中的数据倾斜问题？
1. 观察任务执行时间
 - Spark UI中查看Stage的任务执行时间分布
 - 如果某些任务执行时间远大于平均水平(如10倍以上)
2. 检查Shuffle数据量
 - 查看Spark UI中的Shuffle Read Size
 - 观察各任务读取数据量是否严重不均
3. 分析Key分布：
 - 对原始数据或抽样数据统计Key的分布情况
 - 采样分析: rdd.map(x => (x._1, 1)).countByKey()

如何处理数据倾斜：
1. 预聚合+二次聚合

2. 随机前缀+扩容

3. 使用广播变量优化Join：

4. 使用Spark SQL的AQE功能

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
### Flink状态后端(State Backend)是什么？
1. MemoryStateBackend：适合测试环境，存储小状态
 - 状态存储在JobManager的内存
 - checkpoint存储在JobManager的内存
2. FsStateBackend
 - 状态存储在TaskManager内存或堆外内存
 - checkpoint存储在外部文件系统(HDFS)
3. RocksDBStateBackend:适合生产环境，存储大的状态
 - 状态存储在TaskManager本地RocksDB
 - checkpoint存储在外部文件系统(HDFS)

---
### ValueState、ValueStateDescriptor
两个都是用于状态管理的组件，可以保存和访问每个键的状态数据。
1. ValueState：是一个接口，表示一个可以存储单个值的状态。比如在计算每个用户的累计积分时，可以使用ValueState<Integer>来存储每个用户的当前积分
 - value():获取当前状态的值。如果之前没有设置过值，则返回null；
 - update(T value):更新状态的值
 - clear():清除当前状态
2. ValueStateDescriptor：是用来创建ValueState实例的描述符类，定义了状态的名字、类型信息。如果要使用状态，首先需要定义一个ValueStateDescriptor，然后通过这个描述符来获取相应的ValueState实例。

---
### WindowFunction、ProcessWindowFunction
都适用于对窗口内数据进行处理的函数。
1. WindowFunction:访问窗口的结果以及一些元数据(窗口的开始和结束时间)。
```java
    /**
 * Evaluates the window and outputs none or several elements.
 *
 * @param key The key for which this window is evaluated.窗口的键
 * @param window The window that is being evaluated.当前正在处理的窗口
 * @param input The elements in the window being evaluated.该窗口中的所有元素
 * @param out A collector for emitting elements.用来输出结果的收集器
 * @throws Exception The function may throw exceptions to fail the program and trigger recovery.
 */
    void apply(KEY key, W window, Iterable<IN> input, Collector<OUT> out) throws Exception;
```
2. ProcessWindowFunction：是一个更强大的接口，提供更多的控制和上下文信息，可以提供窗口执行的信息，包括状态和触发器相关的上下文。
```java
  /**
   * Evaluates the window and outputs none or several elements.
   *
   * @param key The key for which this window is evaluated.
   * @param context The context in which the window is being evaluated.提供窗口状态、当前处理时间、当前watermark值等上下文信息
   * @param elements The elements in the window being evaluated.包含窗口中的所有元素
   * @param out A collector for emitting elements. 用来输出结果的收集器
   * @throws Exception The function may throw exceptions to fail the program and trigger recovery.
   */
  public abstract void process(
          KEY key, Context context, Iterable<IN> elements, Collector<OUT> out) throws Exception;
```

---
### DataStreamSource、DataStream、KeyedStream、WindowedStream
1. DataStreamSource：表示数据流的源头，负责从外部系统(比如Kafka、文件、Socket)读取数据
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
// 从 Kafka 读取
Properties props = new Properties();
props.setProperty("bootstrap.servers", "localhost:9092");
DataStreamSource<String> kafkaStream = env.addSource(
    new FlinkKafkaConsumer<>("topic", new SimpleStringSchema(), props));
```
2. DataStream：通用数据流，表示一个无限的、连续的数据流，是所有流操作的基础，支持各种转换操作(map,filter,keyBy)
3. KeyedStream:按键分区的数据流，将数据流按键进行分区，确保相同键的数据由同一个实例处理
 - 通过KeyBy(event -> event.getUserId()方法得到
 - 后续操作只能应用键控操作(比如window，process),不能直接调用非键控操作(如union)
4. WindowedStream: 表示将无限流按时间或数量划分为有限的"桶"，便于批量处理
 - 窗口创建后，需通过apply、reduce等方法定义窗口内数据的处理逻辑
 - 窗口类型有很多种，比如支持滚动窗口、滑动时间窗口、会话窗口

```java
KeyedStream<MyEvent, String> keyedStream = ...;

// 滚动时间窗口（每 5 分钟一个窗口）
WindowedStream<MyEvent, String, TimeWindow> tumblingWindow = 
    keyedStream.window(TumblingEventTimeWindows.of(Time.minutes(5)));

// 滑动时间窗口（窗口大小 10 分钟，滑动步长 5 分钟）
WindowedStream<MyEvent, String, TimeWindow> slidingWindow = 
    keyedStream.window(SlidingProcessingTimeWindows.of(Time.minutes(10), Time.minutes(5)));

// 会话窗口（根据活动时间动态划分窗口）
WindowedStream<MyEvent, String, TimeWindow> sessionWindow = 
    keyedStream.window(EventTimeSessionWindows.withGap(Time.minutes(10)));
```

> DataStreamSource(源头)->DataStream(通用流)->KeyedStream(按键分区)->WindowedStream(窗口化)

---
### keyBy、process、filter、window、trigger、connect、map、reduce
1. map(MapFunction<T, R> mapper):对每个元素应用函数，转换为新元素

2. filter(FilterFunction<T> filter):过滤元素，保留满足条件的元素

3. keyBy(KeySelector<T, K> key):将数据流按键分区，相同键的元素进入同一分区

4. window(TumblingProcessingTimeWindows.of(Time.days(1), Time.hours(-8))):将键控流划分为窗口
在KeyedStream类里window方法的定义如下，可以看到返回的是WindowedStream：
```java
    public <W extends Window> WindowedStream<T, KEY, W> window(
            WindowAssigner<? super T, W> assigner) {
        return new WindowedStream<>(this, assigner);
    }
```

5. trigger(ContinuousProcessingTimeTrigger.of(Time.seconds(30)))：自定义窗口触发时机
在WindowedStream类里的trigger方法定义如下，
```java
    public WindowedStream<T, K, W> trigger(Trigger<? super T, ? super W> trigger) {
        builder.trigger(trigger);
        return this;
    }
```

6. reduce方法，位于WindowedStream类内：对窗口内元素进行增量聚合(如求和、最大值)
使用代码如下：
```java
windowedStream.reduce((x: EmiBillingInfo, y: EmiBillingInfo) => x.reduce(y), new CacheFilterProcessWindowFunction[EmiBillingInfo, EmiBillingInfo, String, TimeWindow] {
   override protected def transfer(in: EmiBillingInfo): EmiBillingInfo = in

                override protected def extractKey(input: EmiBillingInfo): String = input.redisValue()
            })
```

源代码定义如下：
```java
public <R> SingleOutputStreamOperator<R> reduce(
        ReduceFunction<T> reduceFunction, WindowFunction<T, R, K, W> function) {

    TypeInformation<T> inType = input.getType();
    TypeInformation<R> resultType = getWindowFunctionReturnType(function, inType);
    return reduce(reduceFunction, function, resultType);
}
```

7. process(ProcessFunction<T, R> processFunction):对每个元素处理，输出0个或多个元素
8. connect(DataStream<R> dataStream):连接两个数据流，保留各自类型，可共享状态

> 基础转换(map, filter)

> 分区和窗口(keyBy,window)
 
> 聚合与处理(reduce，process)

> 双流操作(connect)

---
### 什么是Flink窗口，为什么流处理需要窗口？Flink支持哪些类型的窗口？
将无界数据流切分成数据块进行处理，在窗口上执行聚合操作。

1. 滚动窗口Tumbling Window：固定大小，不重叠

2. 滑动窗口Sliding Window：固定大小，可重叠

3. 会话窗口Session Window：活动间隙界定，大小不固定

---
### Watermark生成机制，Watermark是怎么生成的？
1. 周期性生成：按照固定的时间间隔自动生成和发出
2. 标点式生成：有数据流中的特定标记事件触发
3. 自定义生成

---
### watermark如何解决乱序问题
1. 延迟窗口触发：给予迟到数据一定的等待时间
2. 处理迟到数据：通过side output或更新结果处理超过水位线的数据

---
### Flink有哪些时间语义？什么是watermark？如何解决数据乱序问题？
有三种时间语义
1. 处理时间Processing Time：数据被处理时的系统时间
2. 事件时间Event Time：数据实际产生时的时间(数据自带时间戳)
3. Ingestion Time：数据进入Flink系统的时间

> 事件时间是最符合业务语义的，但也面临数据延迟和乱序问题，所以有watermark机制

watermark本质上是一个时间戳标记，表示小于此时间戳的数据应该已经到达，可以安全触发所有截止时间小于等于T的窗口计算

---
### Flink窗口在内部是如何实现的？窗口计算的生命周期是怎样的？
窗口操作通常由以下核心组件协同工作：
1. 窗口分配器(WindowAssigner)：决定元素被分配到哪些窗口
2. 触发器(Trigger):决定何时计算窗口结果
3. 窗口函数(Window Function):定义窗口计算逻辑
4. 移除器(Evictor):可选组件，用于在窗口函数前后移除元素

一个典型窗口操作的生命周期如下：
1. 窗口创建：数据到达时，窗口分配器决定它属于哪个窗口，必要时创建新窗口；
2. 数据累积：数据被添加到对应窗口的状态中
3. 触发计算：当满足触发条件时(如Watermark越过窗口结束时间),触发器启动计算
4. 窗口函数执行：对窗口中累积的数据进行聚合计算
5. 结果输出：计算结果被发送到下游
6. 窗口清理：窗口资源被释放，状态被清除

---
### 滑动窗口和滚动窗口，什么场景下应该选择哪种窗口？
1. 滚动窗口
 - 固定大小、无重叠
 - 每个元素只会分配到一个窗口
 - 适合于周期性聚合计算：如每小时统计、每天汇总
2. 滑动窗口
 - 大小固定、可以重叠
 - 每个元素可能分配到多个窗口
 - 由两个参数定义：窗口大小和滑动间隔
 - 适用于滚动平均类计算，比如"每分钟计算过去5分钟数据"

---
### 数据仓库和数据湖的区别？
数据湖主要存放的是非结构化或半结构化数据。例如log文件、社交媒体数据、传感器数据等。

---
### 什么是ocpx广告？
Yes, I am familiar with OCPX advertising. 
OCPX stands for "Optimized Cost Per Click," which is a type of advertising model that uses machine learning algorithms to optimize the cost per click of an ad campaign. 
With OCPX, the advertiser sets a target cost per click, and the algorithm automatically adjusts the bid for each ad placement based on the likelihood of a click and the value of that click to the advertiser. 
This helps to maximize the return on investment (ROI) of the ad campaign and improve the overall performance of the advertising strategy.

---
### 什么是PID控制
pid是一种竞价广告的出价调整算法，根据广告的roi表现和目标roi的差距，实时微调出价。如果roi偏低，就降价，如果偏高就加价。目标是在不断变化的环境中，让投放始终稳定在理想状态。防止流量波动导致成本超支

---
### 超成本、欠成本、OCPM赔付
系统需要通过算法预估和动态调价，让"实际转化成本"围绕广告主设定的"目标成本"上下波动，但是在冷启动阶段，模型数据少、转化数据回传延迟等问题，导致波动

1. 超成本：实际转化成本>广告主设定的目标成本。
 - 对于广告主：预算浪费、ROI暴跌，尤其对中小商家致命
 - 模型数据少导致平台算法预估偏差
2. 欠成本：实际转化成本小于目标成本
 - 对于平台：优质流量被贱卖，挤压平台利润空间
3. OCPM赔付：广告平台为了减弱广告主投放时对超成本的顾虑，同时为模型波动等问题进行兜底，平台对广告投放前14天，当超成本率大于20%时，对超出成本的部分进行赔付

---
### lookalike

---
### 召回-粗排-精排

---
### 混排
比如在信息流推荐场景下，推荐系统会返回一个推荐列表，广告系统会返回一个广告列表，需要综合考虑推荐和广告的组合结果。

---
### 对oCPM的理解

---
### 明投和暗投
主要区别在于广告主对投放渠道的控制程度；

明投是由广告主通过广告平台自主选择投放渠道，指定广告展示的媒体或关键词展示广告；

暗投是由广告平台选择投放渠道，广告主无法完全掌控实际投放渠道，需要依赖平台的流量分配机制。
