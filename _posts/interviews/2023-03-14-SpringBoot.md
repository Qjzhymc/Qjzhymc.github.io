---
title: SpringBoot面试题
author: Yu Mengchi
categories:
  - Interview
  - 面试知识点
  - 中间件
tags:
  - Interview
  - SpringBoot
---
### 什么是IoC，什么是AOP？
IoC是控制反转的意思，它是一种设计思想，或者说是一种模式。主要描述的是Java开发领域对象的创建以及管理的问题。
控制反转就是指不需要在程序中手动创建对象，而是把对象的实例化和管理的权力交给外部环境，比如Spring框架、IoC容器。

IoC的作用就是双方之间不相互依赖，由第三方容器来管理相关资源，这样的话会带来一些好处：
1、第一个就是对象之间的耦合度或者说依赖程度降低；
2、第二个就是资源变的容易管理；比如用Spring容器提供的话很容易就可以实现一个单例。

### 怎么实现IOC？
IoC是一种设计思想，或者说是设计模式，实现IoC的最常见以及最合理的实现方式就是依赖注入（DI）

DI：依赖注入就是指在创建对象时，Spring容器会自动将该对象所依赖的其他对象注入到它里面。

### 什么是AOP？
AOP是指面向切面编程，
使用@Aspect注解表示一个切面类，再在@Before，@AfterReturning，@AfterThrowing注解对应方法，定义不同的通知类型，定义切面和增强操作。

### 实现AOP的核心原理？
Spring实现AOP的核心原理是动态代理和增强器。

在Spring框架中，可以使用以下方式来实现AOP（面向切面编程）：
1. 基于代理的方式：Spring框架利用动态代理技术，在运行时为目标对象生成一个代理对象，并将切面逻辑织入到代理对象的方法执行过程中。通过配置切面和通知（Advice），以及指定切入点（Pointcut）来定义横切关注点的位置。常见的基于代理的AOP实现有：
JDK动态代理：通过接口进行代理，要求目标对象实现接口。
CGLIB动态代理：通过生成子类的方式进行代理，无需目标对象实现接口。
2. 基于字节码增强的方式：Spring框架使用字节码增强技术，将切面逻辑直接织入到目标对象的字节码中，在目标对象的方法执行时插入切面代码。常见的基于字节码增强的AOP实现有：
AspectJ：是一个功能强大且独立的AOP框架，它提供了更加灵活和细粒度的切面编程能力。Spring框架也提供了对AspectJ的集成支持。
3. 声明式AOP：Spring框架还支持通过声明式的方式来实现AOP，即通过配置（如XML配置或注解）来定义切面、通知和切入点。在声明式AOP中，无需直接与AOP框架的API打交道，通过配置可以灵活地指定切面逻辑和连接点。

无论采用哪种方式，实现AOP的关键是定义切面（Aspect）、通知（Advice）和连接点（Pointcut）。切面定义了横切关注点的行为，通知定义了切面在何时执行，连接点定义了切面将被织入到哪些方法上。

需要注意的是，AOP旨在提供横切关注点的复用和集中管理，常见的应用场景包括日志记录、性能监测、事务处理等。在使用AOP时，建议根据具体需求和业务场景选择合适的切面技术

### SpringBoot是如何启动的？启动流程
IOC容器的本质就是初始化BeanFactory和ApplicationContext，

1. 加载配置文件：首先，SpringBoot会加载所有的配置文件，包括application.properties和application.yml等。初始化服务器端口、数据库连接
2. 接着，SpringBoot会扫描所有的类，查找带有@SpringBootApplication注解的类。
3. 创建应用上下文：然后，SpringBoot会创建一个Spring应用程序上下文，该上下文包含了所有的Bean定义。管理所有的bean和它们的依赖关系。
4. SpringBoot会自动配置所有的Bean，包括数据源、Web服务器、日志等。
5. 最后，SpringBoot会启动Web服务器，监听HTTP请求。
6. 运行应用：处理请求

在启动过程中，SpringBoot会自动配置很多东西，大大简化了开发者的工作。

### Spring的自动装配原理？
Spring的自动装配原理是通过扫描应用上下文中的bean定义，自动将符合条件的bean注入到需要依赖它们的bean中。Spring的自动装配有三种方式：

1. 根据名称自动装配：Spring会自动查找与需要依赖的bean名称相同的bean，并将其注入到需要依赖它的bean中。

2. 根据类型自动装配：Spring会自动查找与需要依赖的bean类型相同或是其子类的bean，并将其注入到需要依赖它的bean中。

3. 根据注解自动装配：Spring会自动查找被特定注解修饰的bean，并将其注入到需要依赖它的bean中。

在进行自动装配时，Spring会根据bean的依赖关系自动确定注入顺序，并在需要时执行类型转换。
如果存在多个符合条件的bean，则需要使用@Qualifier注解来指定具体使用哪个bean进行注入。

条件化配置（Conditional Configuration）：根据条件选择性地加载和应用配置
自动配置类：是一种特殊的类，
SpringBootStarter：
类路径扫描和条件触发：
自定义自动配置：


### SpringBoot常用的注解有哪些？讲讲原理？

### 将一个类声明为bean的注解有哪些？
- @Component
- @Repository
- @Service
- @Controller

### 注入一个bean的注解有哪些？
- @Autowired
- @Inject
- @Resource

> @Autowired注解先通过类型匹配，再通过名称匹配，最好结合@Qualifier注解指定名称。

> "注入"的意思就是声明一个对象，类似于 B b = new B(); 的过程，因为对象是直接从Spring容器中取过来的，所以叫"注入"，不用jvm自己创建。

### @Resource 和 @Autoware注入有什么区别？
@Resource 和 @Autowired 是用于依赖注入的注释。它们的主要区别在于：
1. 来源不同：@Resource 是来自于Java EE规范的注释，而@Autowired 是Spring框架提供的注释。
2. 自动装配方式不同：@Resource 默认按照名称进行自动装配，如果找不到对应名称的bean，则会按照类型进行匹配。而@Autowired 默认按照类型进行自动装配，如果找不到对应类型的bean，则会报错。
3. 可以注入的类型不同：@Resource 可以注入任意类型的bean，而@Autowired 只能注入Spring容器中存在的bean。
4. 配置方式不同：@Resource 需要指定name或者type属性来指定要注入的bean，而@Autowired 可以通过@Qualifier注释来指定要注入的bean的名称。

### Bean的生命周期？怎么创建bean的？
Bean的生命周期包括以下阶段：
1. 实例化：当Spring容器启动时，它会创建所有配置的Bean实例。
2. 属性注入：在实例化Bean后，Spring容器将会注入Bean的属性。
3. 初始化：在Bean的属性注入完成后，Spring容器会调用Bean的初始化方法。
4. 使用：此时Bean已经可以被其他对象使用。
5. 销毁：当Spring容器关闭时，它会销毁所有的Bean实例，同时调用Bean的销毁方法。

需要注意的是，Bean的生命周期可以通过实现Spring的Bean生命周期接口来自定义。
例如，可以在Bean的初始化方法中执行一些特定的操作，或者在销毁方法中释放资源。

主要分为三个步骤：
1. 第一个是实例化，会调用createBeanInstance方法，就是new一个对象
2. 第二个就是属性注入，会调用populateBean方法，为new出来的对象填充属性
3. 第三个就是初始化，会调用initializeBean方法，会执行aware接口中的方法，初始化方法，完成AOP代理。

### Spring如何解决循环依赖的？
如果有两个或两个以上的对象互相依赖对方，形成闭环就会出现循环依赖。就会导致对象的创建过程产生死循环。
那在Spring中是通过使用三级缓存来解决的。
一级缓存里存放的是成品对象，成品对象的实例化和初始化都完成了，应用中可以使用的对象都是一级缓存里的。
二级缓存里存放的是半成品对象，用来解决对象创建过程中的循环依赖问题。
三级缓存里存放的是ObjectFactory，用来处理存在AOP时的循环依赖问题。

### 说一下Spring中事务传播行为？
当事务方法被另一个事务方法调用的时候，必须指定事务应该如何传播，比如方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。
所以事务传播行为是为了解决业务层方法之间互相调用的事务问题。
在Spring中有设置不同的事务传播行为：
1、第一个是TransactionDefinition.PROPAGATION_REQUIRED,这个是@Transactional注解默认使用的事务传播行为。
表示如果当前存在事务，则加入该事务，如果当前没有事务，则创建一个新的事务。
2、第二个是TransactionDefinition.PROPAGATION_REQUIRES_NEW,会创建一个新的事务，如果当前存在事务，则把当前事务挂起。
3、第三个是TransactionDefinition.PROPAGATION_NESTED,如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行，
如果没有事务，则创建一个新事务。
4、第四个是TransactionDefinition.PROPAGATION_MANDATORY,如果当前存在事务，则加入该事务，如果当前没有事务，则抛出异常。

### 说一下Spring中事务隔离级别有哪几种？
Spring定义了一个Isolation枚举类表示不同的隔离级别。
有
1、TransactionDefinition.ISOLATION_DEFAULT:表示使用后段数据库默认的隔离级别，比如MySQL就是默认采用REPEATABLE_READ隔离级别。
2、TransactionDefinition.ISOLATION_READ_UNCOMMITTED:表示允许读取尚未提交的数据变更，可能会导致脏读、幻读、不可重复读
3、TransactionDefinition.ISOLATION_READ_COMMITTED:表示允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读和不可重复读还是有可能发生。
4、TransactionDefinition.ISOLATION_REPEATABLE_READ:表示对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止
脏读和不可重复读，但幻读还是有可能发生。
5、TransactionDefinition.ISOLATION_SERIALIZABLE:是最高的隔离级别，所有事务一次逐个执行，这样事务之间就完全不可能产生干扰，这种级别
可以防止脏读、不可重复读、和幻读。但是也会影响到程序的性能。

### 如何理解各种Context上下文？

### 观察者模式很重要？
Spring的事件驱动模型：
有三种角色：事件、事件监听者、事件发布。
只要定义好这三种角色就行，每种角色只需要实现接口就行，最后发布事件发布需要用到上下文的publishEvent(事件)方法。

### 适配器模式怎么工作的？

### @Transactional的原理

