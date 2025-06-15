---
title: xxl-job和DolphinScheduler面试题
author: Yu Mengchi
categories:
  - Interview
  - 面试知识点
  - 中间件
tags:
  - Interview
  - xxljob
---

---
### 分布式调度平台
单体服务实现任务调度：可以通过多线程、Timer类、Spring Task的@Scheduled注解来实现任务调度。

但是在分布式系统中，一个服务有多个实例，通过上面的方式会存在一些问题：
1. 避免任务重复提交；
2. 对任务执行情况统一管理；
3. 增加服务实例或者有服务实例宕机，会影响任务执行。

所以出现了一些分布式任务调度平台，比如xxl-job，DolphinScheduler，Quartz、elastic-job等。

---
### xxl-job的架构设计
模块划分：
1. 调度中心admin：主要负责一些管理工作，比如调度任务的管理、执行器管理、日志管理，会按照调度任务的配置发出调度请求。
2. 执行器executor：负责接收admin的调度请求，执行任务。
 - executor里面包含自定义的各种jobHandler，也就是我们自己的业务代码。每个任务具体怎么执行，就是有jobHandler里的代码逻辑决定的。
 - 执行器里面有两个任务执行的线程池，接收到admin的调度请求后，会把任务放到线程池中，路由到具体的jobHandler中执行。

admin是中心式的，executor是分布式的，支持集群部署，admin可以管理多个executor。

xxl-job的特性
1. 触发策略：提供丰富的任务触发策略，包括：Cron触发、固定间隔触发、固定延时触发、API（事件）触发、人工触发、父子任务触发
2. 调度过期策略：调度中心错过调度时间的补偿处理策略，包括：忽略、立即补偿触发一次等
3. 阻塞处理策略：调度过于密集执行器来不及处理时的处理策略，策略包括：单机串行（默认）、丢弃后续调度、覆盖之前调度
4. 任务超时控制：支持自定义任务超时时间，任务运行超时将会主动中断任务
5. 路由策略：执行器集群部署时提供丰富的路由策略，包括：第一个、最后一个、轮询、随机、一致性HASH、最不经常使用、最近最久未使用、故障转移、忙碌转移等；
6. 分片广播任务：执行器集群部署时，任务路由策略选择”分片广播”情况下，一次任务调度将会广播触发集群中所有执行器执行一次任务，可根据分片参数开发分片任务；
7. 一致性：“调度中心”通过DB锁保证集群分布式调度的一致性, 一次任务调度只会触发一次执行；

xxl-job任务执行流程：
1. 首先executor会向admin自动注册，也可以在admin执行器管理页面手动添加。
2. admin会根据调度器的配置，向executor发送任务的调度请求。把任务的配置信息、参数都发给executor
3. executor接收到调度请求后，获取线程、获取jobHandler，在线程里执行jobHandler逻辑。
4. 在executor中还会写入日志文件、回调执行结果给admin

---
### admin和executor之间是如何通信的
- xxl-job基于netty实现的rpc，admin里有executor的地址，会向executor发送http请求。
- executor基于netty会创建一个tcp服务端，绑定到指定端口，会监听admin的请求，
- 一旦收到admin的请求，会解析请求信息，解析是run接口还是beat接口还是查看log接口。


---
### xxl-job的任务调度器是如何实现的
基于Quartz实现的

---
### 任务失败如何排查
1. 首先xxljob的日志管理页面有调度结果和执行结果的日志，可以判断是调度有问题还是调度成功了但是执行失败了。
2. 如果调度失败，可能是执行器服务异常了
3. 如果执行失败了，可以再日志管理里点开日志文件查看日志，这个日志文件也是在执行器里打印出来的，

---
### DolphinScheduler的架构设计
Apache DolphinScheduler是一个分布式和可扩展的工作流协调平台，具有强大的DAG可视化界面

特性：
1. 去中心化，多master，多worker，避免但master压力过大。
2. 工作流
3. 任务类型多
4. 扩展插件

使用：
1. 创建工作流，在画布上拖上任务节点，配置任务节点，连接节点。

---
### DolphinScheduler上扩展新的任务插件
基于 SPI 开发新插件，可以扩展任务类型、数据源类型；
1. DolphinScheduler dao 层只支持 mysql 和 postgresql，扩展了一个新的达梦数据源；
2. dolphin内置有20多种任务类型，新扩展了自定义的一个任务类型task。

开发 SPI 插件流程：
1. 首先框架本身已经有了一个服务接口，就是TaskChannelFactory接口；
2. 在worker启动时，在TaskPluginManager中，加载所有遵循了SPI规则的插件，
3. 加载流程是，根据服务接口TaskChannelFactory接口类型，用ServiceLoader.load(spiClass)获取所有实现了该接口的类，也就是获取所有插件的Factory类，通过Factory类创建TaskChannel(然后通过TaskChannel创建Task)
4. 所以如果需要新建一个插件，需要插件模块底下新增一个新插件的模块，然后对应的xxxTaskChannelFactory类，xxxTaskChannel类。然后需要把xxxTaskChannelFactory类路径写到META-INF/services/目录文件里，文件名字是接口的全限定名，里面的内容是实现类的全限定名。这样就可以找到这个插件。

SPI：

- 把服务接口和具体的服务实现分开，可以扩展。符合开闭原则，对扩展开放，对修改关闭。

- ServiceLoader是java提供的一种加载服务实现的工具

- 能够通过ServiceLoader.load(xxx.Class)加载的机制是通过Thread Context ClassLoader，也就是线程上下文类加载器，通过这个上下文加载器加载类，负责加载classpath上的类

---
### DolphinScheduler中master和worker之间的通信
基于 netty 实现 rpc 的流程：
1. 遍历所有接口，判断这个 bean 是否有 @RpcService 注解;
2. 再遍历这个 bean 的所有方法，找到带有 @RpcMethod 注解的方法;
3. 将该方法的 invoker 注册到 netty 服务器的 map 中。
4. 之后启动 netty 之后，创建服务端，绑定端口，接收到 rpc 的请求，会从 netty 的 channelHandler 中找到对应的方法，通过反射执行方法。
5. 最后将执行结果返回发送方。

