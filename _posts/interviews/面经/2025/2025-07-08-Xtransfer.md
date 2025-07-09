---
title: XTransfer面试题
author: Yu Mengchi
categories:
  - Interview 
  - 面经
tags:
  - Interview
  - 面试
---

---
### 如何设计一个200万用户登录的登录系统？JWT？

---
### Cookie和Session
1. cookie保存用户信息、偏好设置、购物车信息，存放在浏览器的文件中
2. Session保存认证状态、用户角色等敏感信息，保存在服务端，session id通过cookie传给浏览器保存

---
### JWT？
是Json Web Token，用来登录和认证的。服务端生成JWT，发送给浏览器，浏览器保存在localStorage中，之后每次发送请求都带上JWT，服务端之后接收到请求后会验证JWT，因为JWT里有用私钥加密的signature签名信息，所以可以验证JWT是否被篡改。
1. header
2. payload
3. signature

---
### CSRF？


---
### 购物车结算


---
### 支付的时候如何防止超卖？并发的时候怎么保证库存数量？提交订单的时候库存突然不够了怎么处理？支付的时候价格是从数据库查的还是前端传的？


---
### 数据库表加锁是怎么加的？


---
### FOR UPDATE是加锁是加在索引上吗


---
### RC隔离级别下如何实现ACID
1. A:原子性：WAL写前日志，所有事务更改先记录到日志文件中，再应用到实际的数据页中，如果事务失败，回滚日志
2. C一致性：
3. I隔离性：行级锁，排他锁，MVCC
4. D持久性：写入磁盘

---
### insert语句的执行流程？
连接器 -> 解析器 -> 优化器 -> 写入内存中，记录redo log，进入prepare状态 -> 记录binlog -> redo log提交状态 -> 更新完成

---
### WAL

---
### CommandLineRunner用过吗？
SpringBoot提供的接口，在Spring应用启动完成后执行一些自定义的代码逻辑，做一些初始化操作、数据预加载等。有一个run方法。可以用@Order区分顺序

---
### redis如果hgetall会发生什么？
会发生阻塞，建议使用hscan，可以分批获取数据

![img.png](../../assets/img2/img.png)

---
### Spark比MapReduce快的原因？
1. Spark支持内存计算，MapReduce只能从磁盘读取数据
2. 可以保留中间状态

---
### Spark Streaming和Flink有啥区别？
1. Spark Streaming是微批处理，Flink是无界数据流处理，是实时处理
2. Flink有状态管理，支持多种状态后端RocksDB，可以维护状态
3. Flink有滑动窗口、滚动窗口、会话窗口，可以处理迟到数据

---
### Kettle是如何开发的？
1. 构建DAG图，用拓扑排序，找到根节点。
2. 从根节点开始执行，执行完之后，把后向任务加入带执行队列里
3. 下一步执行所有待执行任务，执行时获取前向依赖节点的输出作为当前节点的输入。

---
### Spring Boot和Spring Framework 区别？
SpringBoot在Spring Framework基础之上的一个框架，简化spring应用的创建过程，约定优于配置。
Spring Framework需要进行大量的XML或注解配置来初始化上下文环境。Spring Boot自带嵌入式服务器。

---
### 多级缓存，本地+分布式缓存？



