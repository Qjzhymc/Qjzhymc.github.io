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

### spring中的IOC和DI是什么？他们的优点是什么？怎么实现IOC？
1. IoC是控制反转的意思，它是一种**设计思想，或者说是一种模式**。指的是把对象的**实例化和管理**的权力交给Spring框架。不需要在程序中手动创建对象。
2. DI依赖注入是IOC的一种实现方式，是指在创建对象的时候，**由spring容器自动将该对象所依赖的其他对象注入进来**，而不是由对象自己创建。

优点是：
1. 第一个就是**对象之间的耦合度或者说依赖程度降低**，对象之间不需要直接相互引用；
2. 第二个就是**资源变的容易管理**。

实现IoC的方式：就是依赖注入（DI）

---
### **spring中的AOP是什么？AOP使用场景？如何实现AOP？AOP核心原理？**
AOP是spring框架的一个很重要的特性，叫面向切面编程。**可以在程序运行时动态地将代码插入到原有代码的流程中，实现功能上的扩展和优化。**
AOP会将一些通用的功能抽取出来封装到一个可重用模块，这个可重用模块就叫切面，然后以切面的形式应用到不同的对象上，**可以避免在每个对象中都重复编写相同的代码，降低模块之间的耦合度**。

**AOP使用场景：**
1. 日志记录
2. 事务管理
3. 性能检测

**如何自己实现AOP：**
1. 首先用@Aspect注解定义你的切面类；
2. 然后在类里用@Pointcut注解定义切面，可以指向自定义的注解，匹配所有自定义注解的方法；
3. 然后再定义通知操作方法，用@Before、@AfterReturning、@AfterThrowing注解定义相应的通知逻辑。
4. 这样的话，所有使用了自定义注解的方法，在执行前、执行后、或者出现异常的时候，都会执行这个切面类的对应的通知逻辑。

> 无论采用哪种方式，实现AOP的关键是定义切面（Aspect）、通知（Advice）和连接点（Pointcut）。切面定义了横切关注点的行为，通知定义了切面在何时执行，连接点定义了切面将被织入到哪些方法上。

**AOP实现的原理：动态代理和增强器。**
1. 基于代理的方式：Spring框架利用动态代理技术，在运行时为目标对象生成一个代理对象，并将切面逻辑织入到代理对象的方法执行过程中。通过配置切面和通知（Advice），以及指定切入点（Pointcut）来定义横切关注点的位置。
 - JDK动态代理：通过接口进行代理，要求目标对象实现接口，使用Proxy.newProxyInstance()创建代理对象。
 - CGLIB动态代理：通过生成子类的方式进行代理，无需目标对象实现接口。
2. 基于字节码增强的方式：Spring框架使用字节码增强技术，将切面逻辑直接织入到目标对象的字节码中，在目标对象的方法执行时插入切面代码。基于字节码增强的AOP实现有：
 - AspectJ：是一个功能强大且独立的AOP框架，它提供了更加灵活和细粒度的切面编程能力。Spring框架也提供了对AspectJ的集成支持。
3. 声明式AOP：Spring框架还支持通过声明式的方式来实现AOP，即通过配置（如XML配置或注解）来定义切面、通知、切入点。在声明式AOP中，无需直接与AOP框架的API打交道，通过配置可以灵活地指定切面逻辑和连接点。

---
### 不用代理模式怎么实现AOP？
- 字节码增强
- 声明式AOP

---
### 

---
### 静态代理和动态代理的区别？静态代理怎么写？动态代理怎么写？
1. 静态代理的代理类**在编译期手动编写**，动态代理的代理类是**在运行时动态生成的**；
2. 静态代理一般是通过实现相同的接口创建代理类。动态代理可以基于接口也可以基于类创建代理类；
3. jdk动态代理是通过实现InvocationHandler接口创建对应的handler类，然后通过Proxy.newProxyInstance创建代理类。
4. 静态代理是，先创建一个目标类，再创建另一个代理类，两个类实现同一个接口，但是代理类的构造函数可以传入一个目标类作为参数，在代理类里先执行自定义操作，再调用目标类的对应方法，实现代理的逻辑

JDK 动态代理怎么创建？
1. 首先创建接口
2. 然后创建InvocationHandler实现类，在invoke()方法中处理方法调用，可以在这里添加增强逻辑
3. 然后使用的时候，可以通过Proxy.newProxyInstance(classLoader, class<>[], handler)创建代理对象，通过代理对象调用方法。

---
### Spring为什么有两种不同的动态代理？jdk动态代理和cglib动态代理
1. JDK动态代理要求目标对象至少实现一个接口，代理类会实现相同的接口，在运行时会将调用转发给实际的目标对象。
2. CGLIB是通过生成目标类的一个子类来实现代理，所以目标对象可以不实现任何接口。

> 使用时，最好实现接口，使用JDK动态代理，更符合java标准。

---
### **Spring的自动装配原理？**
Spring的自动装配原理是通过扫描应用上下文中的bean定义，自动将符合条件的bean注入到需要依赖它们的bean中。
Spring的自动装配有三种方式：
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
1. 启动类会用的@SpringBootApplication，
2. 定义Bean的有@Configuration，@Bean，@Component
3. 依赖注入的有@Autowired，@Qualifier，
4. web开发的有@RestController，@Service，@GetMapping、@PostMapping。还有@Entity实体类
5. @Transactional表示方法需要事务管理，方法执行成功提交事务，发生异常回滚事务。
6. 还有@Value注入配置文件的属性值，@ConfigurationProperties批量绑定配置文件属性到java对象
7. 还有@Scheduled根据注解中的配置定期执行方法，@Async异步执行，方法放入线程池中异步执行。

---
### @Data注解作用于哪个阶段
在编译时，动态生成getter，setter，toString，equals，hash方法。减少样板代码。

> 编译期(lombok) -> 类加载期(修改字节码) -> 运行期(反射或动态代理)

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
### @Resource 和 @Autowired 注入有什么区别？
@Resource 和 @Autowired 是用于依赖注入的注释。它们的主要区别在于：
1. 来源不同：@Resource 是来自于Java EE规范的注释，而 @Autowired 是Spring框架提供的注释。
2. 自动装配方式不同：@Resource 默认按照名称进行自动装配，如果找不到对应名称的bean，则会按照类型进行匹配。而 @Autowired 默认按照类型进行自动装配，如果找不到对应类型的bean，则会报错。
3. 可以注入的类型不同：@Resource 可以注入任意类型的bean，而 @Autowired 只能注入Spring容器中存在的bean。
4. 配置方式不同：@Resource 需要指定name或者type属性来指定要注入的bean，而 @Autowired 可以通过 @Qualifier 注释来指定要注入的bean的名称。

---
### **SpringBoot是如何启动的？启动流程**
1. 启动类有一个 @SpringBootApplication 注解，包含三个核心注解@SpringBootConfiguration表示启动类为配置类，@EnableAutoConfiguration通过内部@Import注解AutoConfigurationImportSelector.class实现自动装配，@ComponentScan默认扫描当前目录及子目录下的bean。
2. 启动的时候会执行SpringApplication的run方法。
- 判断是否是web环境，启动对应的ApplicationContext上下文环境。
- 初始化 Environment 和各种 ApplicationListener 监听器，可以监听启动过程中的各种事件
- 打印banner
- 加载配置类，触发自动配置，核心实现是在AutoConfigurationImportSelector类中，会从spring.factories文件中读取自动配置类，然后加载相应的自动配置。
- 然后再执行prepareContext、refreshContext方法，在refreshContext方法里，会调用refresh方法，会加载BeanDefination、注册Bean、初始化Bean、最后加载 Bean 到 BeanFactory 中。也会执行Bean生命周期的一些回调方法，钩子方法。
3. 启动内嵌的 web 服务器tomcat
4. 执行一些回调方法，ApplicationRunner、CommandLineRunner接口的方法，
5. 最后发送一个启动完成的事件，通知所有监听器应用启动完成，可以接收请求了。

---
### **Bean的生命周期？怎么创建bean的？bean的创建过程？**
Bean的生命周期包括以下阶段/创建Bean的步骤：
1. **实例化**：首先是实例化，当Spring容器启动时，会根据配置创建所有Bean实例。**会调用createBeanInstance方法**，就是new一个对象
2. **属性填充**：然后是属性填充，**Spring会通过依赖注入设置Bean的属性值。会调用populateBean方法**，为new出来的对象填充属性
3. **初始化**：然后是初始化，**会调用initializeBean方法，会执行aware接口中的方法，可以执行一些自定义的初始化逻辑**，完成AOP代理。
 - 可以实现InitializingBean接口的afterPropertiesSet方法
 - 还可以使用@PostConstruct注解
4. **使用**：初始化之后就可以使用了，Bean已经可以被其他对象使用。
5. **销毁**：Spring容器关闭时，它会销毁所有的Bean实例，同时调用Bean的销毁方法。销毁阶段可以执行一些自定义的清理逻辑。
 - 可以实现DisposableBean接口的destroy方法
 - 还可以使用@PreDestroy注解

---
### Bean的几种回调机制
1. @PostConstruct
2. 实现InitializingBean接口的afterPropertiesSet方法
3. 在@Bean(init-method="xxx")中指定初始化方法
4. 实现BeanPostProcessor接口的postProcessBeforeInitialization方法和postProcessAfterInitialization方法，postProcessBeforeDestroy方法和postProcessAfterDestroy方法。

> 只有BeanPostProcessor接口的那几个方法是全局回调的，也就是所有Bean在初始化或销毁时都会执行。

---
### Spring中bean的作用域有哪些? 作用是什么？如何配置bean的作用域？Spring Bean的线程安全问题？
1. singleton：单例，只有一个bean实例，适用于工具类、配置类这种无状态的bean；
2. prototype：每次请求都会创建一个新的bean实例，适合有状态的bean；
3. request：每个http请求都会创建一个新的bean实例，只在web应用中有效，适合与单个http请求相关的bean；
4. session：每个http会话都会创建一个新的bean实例，只在web应用中有效，适合与用户会话相关的bean；
5. global session：每个全局会话都会创建一个新的bean实例。

> 作用域的作用：决定了spring容器如何管理和共享bean的实例。

> 如何配置：用@Scope("prototype")注解配置bean的作用域。

---
### Spring如何解决循环依赖的？
如果有两个或两个以上的对象互相依赖对方，形成闭环就会出现循环依赖。就会导致对象的创建过程产生死循环。

1. Spring如何解决循环依赖？
那在Spring中是通过使用**三级缓存**来解决的。当创建一个Bean时，Spring首先会实例化这个Bean，并将其放入“半成品”缓存中（二级缓存）。然后，在**执行属性填充或setter方法调用**时，如果发现需要注入的Bean尚未完全初始化但已经被实例化（存在于缓存中），Spring就会使用该Bean的当前状态（可能是部分初始化的）来打破循环依赖链。
- 一级缓存是singletonObjects: 一级缓存里存放的是已经完成实例化和初始化的完整单例bean，应用中可以使用的对象都是一级缓存里的。
- 二级缓存是earlySingletonObjects: 二级缓存里存放的是半成品对象，已经实例化，但是还没有完成属性注入，还没有完成初始化的Bean。
- 三级缓存是singletonFactories: 三级缓存里存放的是bean的工厂对象。用于生成早期bean实例。

> 三级缓存通过延迟暴露和动态代理机制，解决循环依赖问题

2. 解决过程是这样的：
- 首先创建bean A：首先将Bean A的工厂对象放到三级缓存；
- 然后提前暴露bean A的引用：如果A依赖B，Spring会从三级缓存获取A的工厂对象，生成一个没有完全初始化的A实例，放到二级缓存；
- 然后开始创建B，如果B依赖A，Spring会从二级缓存中获取提前暴露的A引用；
- 然后B就先初始化完成了，放到一级缓存；
- 最后A继续完成初始化，已经可以获取B了。这样两个bean都创建完成。

3. 什么样的情况没法用三级缓存解决循环依赖？
- **构造器注入**时，无法解决循环依赖。因为构造器注入要求bean必须在创建的时候就**完全初始化完成**，这个就会出现问题，Spring解决不了。
- Spring只能解决**单例作用域的bean**，如果是**prototype原型作用域**的bean，因为原型作用域的bean，每次**请求**都会创建一个**新的bean实例**，所以没法提前暴露引用，Spring也解决不了。

4. 如何避免循环依赖？
- 重构代码：比如可以引入一个新的服务类来解耦两个相互依赖的类；
- 尽量使用setter方法注入依赖，而不是构造器注入。这样可以用Spring自身的三级缓存机制解决。
- 或者使用@Lazy注解延迟加载，避免在bean初始化的时候立即创建依赖。

---
### **spring中的事务管理是什么？它的作用是什么？如何实现？ @Transactional原理是什么？@Transactional注解有哪些属性？**
事务管理可以确保**一组**数据库操作要么全部成功，要么全部失败。保证数据的**一致性**。

作用：
1. 保证数据一致性；
2. spring的事务是用注解声明的，简化了事务处理逻辑；
3. 支持分布式事务，可以解决跨服务的事务一致性问题。

如何实现事务：
在需要事务管理的类或方法上添加@Transcational注解。

@Transactional原理：
底层是通过AOP和TransactionInterceptor实现事务管理的。**基于AOP会生成代理对象，代理对象的方法调用前后添加事务管理逻辑。**

注解的属性有：
1. Propagation: 指定**事务的传播行为**，默认为REQUIRED。传播行为是指当事务方法被另一个事务方法调用的时候，必须指定事务应该如何传播，比如方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。所以事务传播行为是为了解决方法之间互相调用的事务问题。
 - REQUIRED: 如果当前存在事务，则加入该事务，否则创建一个新事务；
 - SUPPORTS: 如果当前存在事务，则加入该事务，否则以非事务方式执行；
 - REQUIRES_NEW: 总是创建一个新事务，如果当前存在事务，则将当前事物挂起。
 - MANDATORY: 如果当前存在事务，则加入该事务，否则抛出异常
 - NOT_SUPPORTED:
 - NEVER:
 - NESTED: 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行，如果没有事务，则创建一个新事务

2. Isolation:指定**事务的隔离级别**，默认为DEFAULT
 - DEFAULT: 表示使用数据库默认的隔离级别，比如MySQL就是默认采用REPEATABLE_READ隔离级别。
 - READ_UNCOMMITTED: 表示允许读取尚未提交的数据变更，可能会导致脏读、幻读、不可重复读
 - READ_COMMITTED: 表示允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读和不可重复读还是有可能发生。
 - REPEATABLE_READ: 表示对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读还是有可能发生。
 - SERIALIZABLE: 是最高的隔离级别，所有事务依次逐个执行，这样事务之间就完全不可能产生干扰，这种级别可以防止脏读、不可重复读和幻读。但是也会影响到程序的性能。

3. RollbackFor: 指定哪些异常会导致事务回滚

---
### 事务的@Transactional注解，A事务调用B事务，当B事务抛出异常之后，在默认事务传播机制下，会出现什么情况？
如果A捕获了异常但是没有处理事务异常状态的话，A事务也会执行失败，因为默认事务传播行为是Required，会共享B方法里的事务，导致事务回滚并抛出异常。

解决方法：
 - 调整传播行为
 - 在处理异常的代码块里显式管理事务状态
 - 设置noRollbackFor={xxx.class}

---
### Spring 事务什么时候会失效？
1. 方法自调用：如果是使用声明式事务，就是在方法上用@Transactional注解的，那它底层其实就是AOP，访问这个方法，是先生成代理类，然后在代理类里访问这个方法的，所以如果是在同一个类里调用加了@Transactional注解的事务方法，
   调用的不是动态代理对象里的方法，而是当前对象的方法，所以就没有事务的逻辑；
2. 异常被捕获：生成的动态代理类会加上trycatch异常逻辑，在catch异常的时候会回滚事务，动态代理类里会处理异常，所以如果**已经在方法里处理了异常，没有抛出来**，那么动态代理类就不知道发生异常，也就不会执行事务回滚的逻辑了。
3. 方法非public: 声明式事务方法只能是public
4. 数据库不支持

---
### BeanFactory和FactoryBean
1. BeanFactory是Spring容器的根接口，负责管理Bean的生命周期，可以通过BeanFactory获取Bean
2. FactoryBean是一种特殊的Bean，用于封装复杂对象的创建逻辑

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
### 如何理解各种Context上下文？

---
### ApplicationContextAware接口？BeanNameAware接口？
当一个对象实现了ApplicationContextAware接口的时候，ApplicationContext在创建这个实例的时候，可以让实例获取ApplicationContext的引用。
> 各种Aware接口就是向实例提供资源的接口。

---
### Spring框架中都用到了哪些设计模式？观察者模式很重要？适配器模式怎么工作的？
Spring的事件驱动模型：
有三种角色：事件、事件监听者、事件发布。
只要定义好这三种角色就行，每种角色只需要实现接口就行，最后事件发布需要用到上下文的publishEvent(事件)方法。

---
### Spring MVC处理流程？

---
### 设计一个容量不超过10的IOC容器(BeanFactory)
