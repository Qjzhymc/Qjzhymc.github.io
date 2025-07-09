---
title: HDFS和MapReduce面试题
author: Yu Mengchi
categories:
  - Interview 
  - 面试知识点
  - 系统设计
tags:
  - Interview
  - 分布式系统
---

---
### HDFS架构是怎样的？
HDFS由一个NameNode和多个DataNode组成。
1. NameNode是主节点，负责管理文件系统的命名空间和数据块的映射关系，
2. DataNode是从节点，负责存储数据块。

namespace是什么？
存储在NameNode节点中，指的是文件和目录的层次结构。

---
### MapReduce运行流程原理？
MapReduce 运行流程：首先将输入数据分片，每个分片由独立 Map 任务处理，Map 按键值对处理数据并输出中间结果；
然后通过 Shuffle 阶段对中间结果排序、分区，将相同键的数据汇聚到同一 Reduce 任务；
最后 Reduce 合并处理相同键的数据并输出最终结果。
过程中由 JobTracker 协调任务分配，TaskTracker 执行具体任务，适用于大规模数据并行计算。


