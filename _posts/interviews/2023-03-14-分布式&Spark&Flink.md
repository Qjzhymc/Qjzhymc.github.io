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
  
# 分布式系统面试题

---

### 有哪些分布式一致算法？分布式一致性协议？

---

### 系统的CAP性质？如何保证AP和CP？

---

### Paxos算法原理讲解一下？

---

### ZooKeeper等应用分布式协议的中间件有哪些？

### 使用Spark有什么需要注意的地方？

1. 内存管理：Spark依赖内存，配置集群的内存设置是很重要的
2. 数据分区：Spark将数据分区到集群的不同的节点上，并行处理，理解数据是怎么分区的，怎么控制分区实现最优性能表现
3. 序列化：Spark使用序列化在集群节点间传输数据。选择合适的序列化格式会影响到性能。
4. 缓存：Spark会缓存数据到内存，理解缓存是怎么工作的，并且什么时候需要使用缓存
5. 资源分配：Spark允许为特定任务分配资源。理解怎么分配资源
6. 调优：Spark里有很多配置项，理解怎么调这些选项。

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

### Spark遇到数据倾斜怎么处理？
1. 重新分区：如果数据集中存在一些热点数据，可以对数据进行重新分区，将热点数据分散到多个分区中，
可以使用repartition()或coalesce()函数对数据进行重新分区。此外，还可以使用随机前缀、哈希等
方法来对数据进行分区，从而避免热点数据集中在同一个分区中的情况。
2. 数据重组
3. 采样：通过采样数据进行分析，可以更好地了解数据的分布情况，识别热点数据并采取相应的处理方法
4. 广播变量：

### 使用随机前缀解决步骤
1. 将数据集按照某个关键字进行分组，例如按照用户id进行分组
2. 对每个分组的关键字进行随机前缀处理，例如在用户id前添加一个随机字符串
3. 将处理后的数据集进行重新分区，使每个分区中的数据量均匀分布
4. 对每个分区中的数据进行聚合操作，例如求和或计数
5. 将每个分区的结果进行汇总，得到最终的结果


### Spark的架构和组件？

### Spark的基本数据结构？RDD、DataFrame和DataSet

### Spark SQL

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
### Spark的部署和集群管理？

### Spark的实战应用

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

### 数据仓库和数据湖的区别？
数据湖主要存放的是非结构化或半结构化数据。例如log文件、社交媒体数据、传感器数据等。

### 什么是ocpx广告？
Yes, I am familiar with OCPX advertising. 
OCPX stands for "Optimized Cost Per Click," which is a type of advertising model that uses machine learning algorithms to optimize the cost per click of an ad campaign. 
With OCPX, the advertiser sets a target cost per click, and the algorithm automatically adjusts the bid for each ad placement based on the likelihood of a click and the value of that click to the advertiser. 
This helps to maximize the return on investment (ROI) of the ad campaign and improve the overall performance of the advertising strategy.
