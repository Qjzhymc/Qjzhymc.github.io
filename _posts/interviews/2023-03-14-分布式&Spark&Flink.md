---
title: 分布式面试题
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
