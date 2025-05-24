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

### spring中的IOC和DI是什么？他们的优点是什么？

1. IoC是控制反转的意思，它是一种设计思想，或者说是一种模式。主要描述的是对象的创建以及管理的问题。
 - 控制反转就是指不需要在程序中手动创建对象，而是把对象的实例化和管理的权力交给Spring框架。

2. DI依赖注入是IOC的一种实现方式，是指在创建对象的时候，由spring容器自动将该对象所依赖的其他对象注入进来，而不是由对象自己创建。
 - 依赖注入的目的是让对象之间保持松耦合的关系。

优点是：
1. 第一个就是对象之间的耦合度或者说依赖程度降低，对象之间不需要直接相互引用；
2. 第二个就是资源变的容易管理。比如用Spring容器提供的话很容易就可以实现一个单例；
3. 可以支持AOP，将一些通用的功能抽取出来，比如日志记录、事务管理，以切面的形式应用到不同的对象上，可以避免在每个对象中都重复编写相同的代码。

---
### 怎么实现IOC？
IoC是一种设计思想，或者说是设计模式，实现IoC的方式就是依赖注入（DI）

DI：依赖注入就是指在创建对象的时候，Spring容器会自动将该对象所依赖的其他对象注入进来。

---
### spring中的AOP是什么？如何实现AOP？

AOP是spring框架的一个很重要的特性，叫面向切面编程。AOP会将一些通用的功能抽取出来，比如日志记录、事务管理，封装到一个可重用模块，这个可重用模块就叫切面，然后以切面的形式应用到不同的对象上，
可以避免在每个对象中都重复编写相同的代码，降低模块之间的耦合度。

spring中的AOP实现的原理主要是基于动态代理来实现的，通过动态代理机制在目标对象的方法调用时插入切面的逻辑。
spring AOP使用了java原生的基于Proxy类的JDK动态代理，还有基于CGLIB的动态代理
1. JDK动态代理：
2. CGLIB动态代理：

自己使用AOP的话，
1. 首先用@Aspect注解定义你的切面类；
2. 然后在类里用@Pointcut注解定义切面，可以指向自定义的注解，匹配所有自定义注解的方法；
3. 然后再定义通知操作方法，用@Before、@AfterReturning、@AfterThrowing注解定义相应的通知逻辑。
4. 这样的话，所有使用了自定义注解的方法，在执行前、执行后、或者出现异常的时候，都会执行这个切面类的对应的通知逻辑。

---
### 实现AOP的核心原理？
Spring实现AOP的核心原理是动态代理和增强器。

在Spring框架中，可以使用以下方式来实现AOP（面向切面编程）：
1. 基于代理的方式：Spring框架利用动态代理技术，在运行时为目标对象生成一个代理对象，并将切面逻辑织入到代理对象的方法执行过程中。通过配置切面和通知（Advice），以及指定切入点（Pointcut）来定义横切关注点的位置。
 - JDK动态代理：通过接口进行代理，要求目标对象实现接口。
 - CGLIB动态代理：通过生成子类的方式进行代理，无需目标对象实现接口。
2. 基于字节码增强的方式：Spring框架使用字节码增强技术，将切面逻辑直接织入到目标对象的字节码中，在目标对象的方法执行时插入切面代码。基于字节码增强的AOP实现有：
 - AspectJ：是一个功能强大且独立的AOP框架，它提供了更加灵活和细粒度的切面编程能力。Spring框架也提供了对AspectJ的集成支持。
3. 声明式AOP：Spring框架还支持通过声明式的方式来实现AOP，即通过配置（如XML配置或注解）来定义切面、通知、切入点。在声明式AOP中，无需直接与AOP框架的API打交道，通过配置可以灵活地指定切面逻辑和连接点。

无论采用哪种方式，实现AOP的关键是定义切面（Aspect）、通知（Advice）和连接点（Pointcut）。切面定义了横切关注点的行为，通知定义了切面在何时执行，连接点定义了切面将被织入到哪些方法上。

需要注意的是，AOP旨在提供横切关注点的复用和集中管理，常见的应用场景包括日志记录、性能监测、事务处理等。

---
### SpringBoot是如何启动的？启动流程
IOC容器的本质就是初始化BeanFactory和ApplicationContext，

1. 加载配置文件：首先，SpringBoot会加载所有的配置文件，包括application.properties和application.yml等。初始化服务器端口、数据库连接
2. 然后，SpringBoot会扫描所有的类，查找带有@SpringBootApplication注解的类。
3. 创建应用上下文：然后，SpringBoot会创建一个Spring应用程序上下文，该上下文包含了所有的Bean定义。管理所有的bean和它们的依赖关系。
4. SpringBoot会自动配置所有的Bean，包括数据源、Web服务器、日志等。
5. 最后，SpringBoot会启动Web服务器，监听HTTP请求。
6. 运行应用：处理请求

在启动过程中，SpringBoot会自动配置很多东西，简化了开发者的工作。

---
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

---

### SpringBoot常用的注解有哪些？讲讲原理？

启动类会用的@SpringBootApplication，

定义Bean的有@Configuration，@Bean，@Component

依赖注入的有@Autowired，@Qualifier，

web开发的有@RestController，@Service，@GetMapping、@PostMapping。还有@Entity实体类

@Transactional表示方法需要事务管理，方法执行成功提交事务，发生异常回滚事务。

还有@Value注入配置文件的属性值，@ConfigurationProperties批量绑定配置文件属性到java对象

还有@Scheduled根据注解中的配置定期执行方法，@Async异步执行，方法放入线程池中异步执行。

---
### 将一个类声明为bean的注解有哪些？
- @Component
- @Repository
- @Service
- @Controller

---
### 注入一个bean的注解有哪些？
- @Autowired
- @Inject
- @Resource

> @Autowired注解先通过类型匹配，再通过名称匹配，最好结合@Qualifier注解指定名称。

> "注入"的意思就是声明一个对象，类似于 B b = new B(); 的过程，因为对象是直接从Spring容器中取过来的，所以叫"注入"，不用jvm自己创建。

---
### @Resource 和 @Autoware注入有什么区别？
@Resource 和 @Autowired 是用于依赖注入的注释。它们的主要区别在于：
1. 来源不同：@Resource 是来自于Java EE规范的注释，而@Autowired 是Spring框架提供的注释。
2. 自动装配方式不同：@Resource 默认按照名称进行自动装配，如果找不到对应名称的bean，则会按照类型进行匹配。而@Autowired 默认按照类型进行自动装配，如果找不到对应类型的bean，则会报错。
3. 可以注入的类型不同：@Resource 可以注入任意类型的bean，而@Autowired 只能注入Spring容器中存在的bean。
4. 配置方式不同：@Resource 需要指定name或者type属性来指定要注入的bean，而@Autowired 可以通过@Qualifier注释来指定要注入的bean的名称。

---

### Bean的生命周期？怎么创建bean的？
Bean的生命周期包括以下阶段：
1. 实例化：首先是实例化，当Spring容器启动时，会根据配置创建所有Bean实例。
2. 属性填充：然后是属性填充，Spring会通过依赖注入设置Bean的属性值。
3. 初始化：然后是初始化，初始化阶段可以执行一些自定义的初始化逻辑。
- 可以实现InitializingBean接口的afterPropertiesSet方法
- 还可以使用@PostConstruct注解
4. 使用：初始化之后就可以使用了，Bean已经可以被其他对象使用。
5. 销毁：Spring容器关闭时，它会销毁所有的Bean实例，同时调用Bean的销毁方法。销毁阶段可以执行一些自定义的清理逻辑。
- 可以实现DisposableBean接口的destroy方法
- 还可以使用@PreDestroy注解

> 还可以在初始化前后插入自定义处理逻辑，使用BeanPostProcessor接口的postProcessBeforeInitialization方法和postProcessAfterInitialization方法

spring创建bean主要分为三个步骤：
1. 第一个是实例化，会调用createBeanInstance方法，就是new一个对象
2. 第二个就是属性注入，会调用populateBean方法，为new出来的对象填充属性
3. 第三个就是初始化，会调用initializeBean方法，会执行aware接口中的方法，初始化方法，完成AOP代理。

---

###Spring中bean的作用域有哪些?作用是什么？如何配置bean的作用域？

1. singleton：单例，只有一个bean实例，适用于工具类、配置类这种无状态的bean；
2. prototype：每次请求都会创建一个新的bean实例，适合有状态的bean；
3. request：每个http请求都会创建一个新的bean实例，只在web应用中有效，适合与单个http请求相关的bean；
4. session：每个http会话都会创建一个新的bean实例，只在web应用中有效，适合与用户会话相关的bean；
5. global session：每个全局会话都会创建一个新的bean实例。

> 作用：决定了spring容器如何管理和共享bean的实例。

> 如何配置：用@Scope("prototype")注解配置bean的作用域。

---

### Spring中的BeanFactory和ApplicationContext有什么区别？

两个都是加载、管理bean的接口, 区别是：
- ApplicationContext是BeanFactory的扩展，除了BeanFactory的功能外，还支持事件监听、资源加载、国际化的功能，功能更强大。
- 另外BeanFactory是延迟加载bean的，bean在第一次被请求的时候才会创建；ApplicationContext是预加载的，在启动的时候就创建所有的bean。

> 创建ApplicationContext实例的时候，通常会指定配置源，比如XML配置文件的位置或者配置类，然后容器根据这些配置信息来启动并初始化整个应用上下文，可以通过ApplicationContext来查找需要的bean。

> 提供依赖注入和bean的生命周期管理的功能。通过读取配置的数据，比如XML文件、注解，容器可以知道需要实例化的对象以及这些对象应该如何被配置。

---

### Spring中的常用接口BeanPostProcessor、BeanFactoryPostProcessor分别是什么？作用是什么？

1. BeanPostProcessor是对bean初始化前后添加自己的逻辑，通过实现这个接口的postProcessBeforeInitialization方法和postProcessAfterInitialization方法，分别在bean初始化之前和之后做自定义逻辑。
2. BeanFactoryPostProcessor是对bean初始化之前修改bean的元信息，比如修改bean的作用域、初始化方法。这个接口有一个postProcessBeanFactory方法，可以在这个方法里修改bean定义。

> 总结就是一个是自定义bean初始化前后的逻辑，一个是在初始化之前修改bean的元信息。
> 一个在实例化之后，一个在实例化之前。

> 加载bean定义->bean实例化->bean初始化

---

### Spring中常用工具类JdbcTemplate、RestTemplate分别是什么？作用是什么？

1. JdbcTemplate是Spring提供的简化JDBC操作的工具类，可以简化数据库连接的获取和关闭。
- 会自动关闭数据库连接；
- 会自动处理sql语句的执行；
- 有很多执行sql的方法，比如query、queryForList、queryForMap、update。
2. RestTemplate是一个用于HTTP访问的工具类，简化了HTTP通信过程。
- 支持各种HTTP方法
- 支持JSON、XML各种数据格式

---

### Spring如何解决循环依赖的？
如果有两个或两个以上的对象互相依赖对方，形成闭环就会出现循环依赖。就会导致对象的创建过程产生死循环。

1. Spring如何解决循环依赖？
那在Spring中是通过使用**三级缓存**来解决的。
- 一级缓存是singletonObjects: 一级缓存里存放的是已经完成实例化和初始化的单例bean，应用中可以使用的对象都是一级缓存里的。
- 二级缓存是earlySingletonObjects: 二级缓存里存放的是半成品对象，存放提前暴露的bean实例，还没有完全初始化完成，用来解决对象创建过程中的循环依赖问题。
- 三级缓存是singletonFactories: 三级缓存里存放的是bean的工厂对象。

2. 解决过程是这样的：
- 首先创建bean A：首先将Bean A的工厂对象放到三级缓存；
- 然后提前暴露bean A的引用：如果A依赖B，Spring会从三级缓存获取A的工厂对象，生成一个没有完全初始化的A实例，放到二级缓存；
- 然后开始创建B：如果B依赖A，Spring会从二级缓存中获取提前暴露的A引用；
- 然后B就先初始化完成了，放到一级缓存；
- 最后A继续完成初始化，已经可以获取B了。这样两个bean都创建完成。

3. 限制，什么时候没法解决循环依赖?
- 构造器注入时，无法解决循环依赖。因为构造器注入要求bean必须在创建的时候就完全初始化完成，这个就会出现问题，Spring解决不了。
- Spring只能解决单例作用域的bean，如果是prototype原型作用域的bean，因为原型作用域的bean，每次请求都会创建一个新的bean实例，所以没法提前暴露引用，Spring也解决不了。

4. 如何避免循环依赖？
- 重构代码：比如可以引入一个新的服务类来解耦两个相互依赖的类；
- 尽量使用setter方法注入依赖，而不是构造器注入。这样可以用Spring自身的三级缓存机制解决。
- 或者使用@Lazy注解延迟加载，避免在bean初始化的时候立即创建依赖。

---

### **spring中的事务管理是什么？它的作用是什么？如何实现？**

事务管理是spring框架的一个特性，可以确保一组数据库操作要么全部成功，要么全部失败。保证数据的一致性。

作用：
1. 保证数据一致性；
2. spring的事务是用注解声明的，简化了事务处理逻辑；
3. 支持分布式事务，可以解决跨服务的事务一致性问题。

如何实现事务：
在需要事务管理的类或方法上添加@Transcational注解。

注解的属性有：
1. propagation:指定**事务的传播行为**，默认为REQUIRED.
- REQUIRED:如果当前存在事务，则加入该事务，否则创建一个新事务；
- SUPPORTS:如果当前存在事务，则加入该事务，否则以非事务方式执行；
- REQUIRES_NEW:总是创建一个新事务，如果当前存在事务，则将当前事物挂起。
- MANDATORY:如果当前存在事务，则加入该事务，如果当前没有事务，则抛出异常
- NOT_SUPPORTED:
- NEVER:
- NESTED:如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行，如果没有事务，则创建一个新事务

2. isolation:指定**事务的隔离级别**，默认为DEFAULT.
- READ_UNCOMMITTED:读未提交；
- READ_COMMITTED:读已提交；
- REPEATABLE_READ:可重复读；
- SERIALIZABLE:串行化。
- DEFAULT:

3. rollbackFor:指定哪些异常会导致事务回滚

---

### 说一下Spring中事务传播行为？
当事务方法被另一个事务方法调用的时候，必须指定事务应该如何传播，比如方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。
所以事务传播行为是为了解决业务层方法之间互相调用的事务问题。

---
### 事务的@Transactional注解，A事务调用B事务抛出异常之后，在默认事务传播机制下，会提交异常，因为rollback异常标志位已经被设置有值


---
### **说一下Spring中事务隔离级别有哪几种？**
Spring定义了一个Isolation枚举类表示不同的隔离级别。
有
1. DEFAULT:表示使用数据库默认的隔离级别，比如MySQL就是默认采用REPEATABLE_READ隔离级别。
2. READ_UNCOMMITTED:表示允许读取尚未提交的数据变更，可能会导致脏读、幻读、不可重复读
3. READ_COMMITTED:表示允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读和不可重复读还是有可能发生。
4. REPEATABLE_READ:表示对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止
脏读和不可重复读，但幻读还是有可能发生。
5. SERIALIZABLE:是最高的隔离级别，所有事务依次逐个执行，这样事务之间就完全不可能产生干扰，这种级别
可以防止脏读、不可重复读、和幻读。但是也会影响到程序的性能。

---
### 如何理解各种Context上下文？

---
### 观察者模式很重要？
Spring的事件驱动模型：
有三种角色：事件、事件监听者、事件发布。
只要定义好这三种角色就行，每种角色只需要实现接口就行，最后事件发布需要用到上下文的publishEvent(事件)方法。

---
### 适配器模式怎么工作的？

---
### @Transactional的原理

---
### ApplicationContextAware接口
当一个对象实现了ApplicationContextAware接口的时候，ApplicationContext在创建这个实例的时候，可以让实例获取ApplicationContext的引用。

> 各种Aware接口就是向实例提供资源的接口。

---
### BeanNameAware接口

ApplicationContext向实例提供BeanName
