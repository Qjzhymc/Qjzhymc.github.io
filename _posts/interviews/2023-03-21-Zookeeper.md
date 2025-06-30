---
title: Zookeeper面试题
author: Yu Mengchi
categories:
  - Interview 
  - 面试知识点
  - 中间件
tags:
  - Interview
  - 微服务
  - Zookeeper
---

---
### zookeeper的核心原理是什么？
zookeeper的核心原理是ZAB协议，它是一种专门为分布式协调设计的原子广播协议。ZAB通过以下机制保证数据一致性。
- 消息广播机制：所有的写请求由leader节点处理，通过两阶段提交（Proposal + commit）广播到集群，确保半数以上节点确认后再提交事务，避免数据不一致；
- 崩溃恢复模式：当leader失效时，ZAB通过选举新leader并同步历史事务日志（ZXID）恢复状态，确保事务顺序性和全局一致性；
- ZXID全局递增：每个事务分配唯一的64位ZXID（高32位为epoch，低32位为事务序号），保证操作顺序和因果性。

---
### ZAB协议流程

---
### zookeeper是做什么的？有什么作用？
- 配置中心：统一存储配置信息，利用watcher机制实时推送变更，避免手动更新；
- 集群管理：通过临时节点监控节点存活状态，自动剔除失效节点；
- 分布式锁：通过有序节点实现公平锁，解决资源竞争问题；
- 命名服务：维护全局唯一的服务地址列表，支持服务发现和负载均衡（如Dubbo）；
- Leader选举：通过ZAB协议快速选出新leader，保障高可用性（如HBase Master选举）。

---
### zookeeper中分为哪几种数据节点（znode）类型？
- 持久节点（persistent）：一旦创建就一直存在，即使zookeeper集群宕机，只能删除
- 临时节点（ephemeral）：临时节点生命周期是和客户端会话绑定的，会话消失节点消失。临时节点只能作为叶子节点，不能创建子节点
- 持久顺序节点（persistent_sequential）:是持久节点，并且它的子节点名称有顺序性
- 临时顺序节点（ephemeral_sequential）：是临时节点，并且子节点名称有顺序性

---
### zookeeper集群如何部署？
集群部署采用主从结构，分为Leader节点、Follower节点、Observer节点。Observer节点不参与Leader节点选举。

- Leader：负责读和写，负责投票的发起和决议，更新系统状态；
- Follower：读，参与选举过程中的投票；
- Observer：读，不参与投票，也不参与"过半写成功"策略。

---
### zookeeper集群为啥最好奇数台？脑裂现象？
首先zoopeeker规定如果集群有机器宕机，那么剩下的机器一定要比宕掉的机器多整个集群才能正常提供服务。

另外集群通常分布式部署，通过网络连接，如果没有选举的过半机制，当集群之间网络出现故障的时候，每个集群内部可能会选举出各自的leader，
导致整个集群出现多个leader，这样就会发生数据一致性问题。有了选举的过半机制之后，机器数少的那些机房是没办法选举出leader节点的，
所以就不会出现脑裂现象。

- 防止由脑裂造成的集群不可用；
- 在容错能力相同的情况下，奇数台更省资源。

---
### zookeeper选举的原理和流程？
ZAB协议 raft协议

zookeeper会在集群初始化的时候或者是leader宕机的时候进行leader选举，选举的过程中会涉及到myid（集群机器id）、zxid（事务id）的比较，集群里每个服务器都会给自己投票，然后再广播给其他服务器
其他服务器比较自己节点信息和其他节点的信息，比较myid和zxid，更新自己的投票，投票会进行很多轮，每轮都统计投票结果，当有超过半数的机器都有相同的投票结果的时候，就说明选举出了leader，否则继续
投票。确定了leader之后，就更新自己的服务器状态，有leading、following、observing。

---
### zookeeper集群中Leader选举过程
- leader election（选举阶段）：只要有一个节点得到超过半数节点的票数，当选准leader；
- discovery（发现阶段）：followers将最近收到的事务提议同步给准leader；
- synchronization（同步阶段）：leader将前一阶段获得的最新的提议历史同步给集群内所有的副本；
- broadcast（广播阶段）：集群对外提供事务服务，leader可以进行消息广播。

---
### ZooKeeper的Leader选举具体过程有哪些？
1. 初始化选举状态
- 每个节点启动时都处于LOOKING状态
- 设置自己的选票(myid, zxid)为初始选票
- zxid代表节点的事务ID，值越大说明数据越新

2. 第一轮投票
- 每个节点先投票给自己
- 将投票信息(myid, zxid)广播给其他节点
- 收到投票后,保存到本地投票箱

3. 选票PK规则
- 优先比较zxid，较大者胜出
- zxid相同时，比较myid，较大者胜出
- 根据PK结果更新自己的投票

4. 统计投票
- 每轮投票后统计是否有超过半数的相同投票
- 若有节点得票超过半数，该节点当选Leader
- 否则进入下一轮投票

5. 选举完成
- 当选Leader的节点状态变更为LEADING
- 其他节点状态变更为FOLLOWING
- Observer节点不参与投票,状态为OBSERVING

举例说明:
假设有3个节点,myid分别为1、2、3,zxid都相同:
1. 第一轮每个节点投自己
2. 收到其他投票后,按照PK规则都会选择myid=3的节点
3. 第二轮投票全部投给节点3
4. 节点3得到超过半数选票,当选Leader

>[ZooKeeper:Wait-free coordination for Internet-scale systems](http://web.eecs.umich.edu/~manosk/assets/slides/f21/zookeeper.pdf)
