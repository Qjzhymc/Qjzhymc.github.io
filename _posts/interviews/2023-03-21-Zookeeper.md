---
title: Zookeeper面试题
author: Yu Mengchi
categories:
  - Interview 
tags:
  - Interview
  - 微服务
  - Zookeeper
---
  
# Zookeeper

---

### zookeeper是做什么的？



---

### zookeeper中分为哪几种数据节点（znode）类型？

- 持久节点（persistent）：一旦创建就一直存在，即使zookeeper集群宕机，只能删除
- 临时节点（ephemeral）：临时节点生命周期是和客户端会话绑定的，会话消失节点消失。临时节点只能作为叶子节点，不能创建子节点
- 持久顺序节点（persistent_sequential）:是持久节点，并且它的子节点名称有顺序性
- 临时顺序节点（ephemeral_sequential）：是临时节点，并且子节点名称有顺序性


---

### zookeeper集群如何部署？


集群部署采用主从结构，分为Leader节点、Follower节点、Observer节点。Observer节点不参与Leader节点选举。

- Leader：负载读和写，负责投票的发起和决议，更新系统状态
- Follower：读，参与选举过程中的投票
- Observer：读，不参与投票，也不参与"过半写成功"策略

---

### zookeeper集群中Leader选举过程

- leader election（选举阶段）：只要有一个节点得到超过半数节点的票数，当选准leader
- discovery（发现阶段）：followers将最近收到的事务提议同步给准leader
- synchronization（同步阶段）：leader将前一阶段获得的最新的提议历史同步给集群内所有的副本
- broadcast（广播阶段）：集群对外提供事务服务，leader可以进行消息广播

---




