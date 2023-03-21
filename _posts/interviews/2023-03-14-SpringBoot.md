---
title: SpringBoot面试题
author: Yu Mengchi
categories:
  - Interview
tags:
  - Interview
  - SpringBoot
---
  
# SpringBoot面试题

---

### SpringBoot是如何启动的？启动流程

---

### Spring的自动装配原理？



---

### SpringBoot常用的注解有哪些？讲讲原理？

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


---

### Bean的生命周期？


---

### 如何理解各种Context上下文？

---

### 观察者模式很重要？

Spring的事件驱动模型：有三种角色：事件、事件监听者、事件发布。只要定义好这三种角色就行，每种角色只需要实现接口就行，最后发布事件发布需要用到上下文的publishEvent(事件)方法。

---

### 适配器模式怎么工作的？

