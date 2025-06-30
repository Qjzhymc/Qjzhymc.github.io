---
title: Java哈希表HashMap
author: Yu Mengchi
categories:
  - Interview 
  - 面试知识点
  - Java基础
tags:
  - Interview
  - Java
---


---
### HashMap和ConcurrentHashMap的实现原理？区别？
1. HashMap底层是**基于数组和链表或者红黑树**实现的，数组的每个位置是一个Entry类型，用于存储键值对，当多个键的哈希值相同时，也就是发生哈希冲突时，
这些冲突的键值对会存储在同一个桶中，形成一个链表，如果链表长度超过一个阈值，默认是8，链表会被转换成红黑树，提高查找效率。 
 - 插入键值对时，先计算键的hash值，然后用hash值与capacity-1做与运算，这个capacity是指数组的容量，得到这个键对应的数组的下标index值。
 - 当发生hash冲突时，也就是说多个键映射到同一个桶中，HashMap会使用链表法解决冲突，在同一个桶中形成链表。如果链表长度超过8，将链表转换成红黑树；如果链表长度减少到6或者更少，将红黑树转换回链表。
 - 扩容机制：当哈希表中的元素数量超过当前容量乘以负载因子(这个负载因子默认是0.75)时，HashMap会触发扩容操作。扩容时，会创建一个新的数组，容量是原数组的两倍，然后将所有键值对重新映射到新数组中。
2. ConcurrentHashMap是线程安全的哈希表，底层结构和HashMap类似，但是为了支持线程安全，采用了**分段锁或CAS操作**。
 - 在JDK早期版本中采用的是**Segment分段锁**，将数组分成多个段，每个段是一个独立的锁，使用分段锁减小锁的粒度，每个分段锁保护数组的一部分。默认有16个segment分段锁，segment是继承自ReentrantLock实现的，
 - 在JDK1.8版本以后，ConcurrentHashMap使用CAS操作来更新数组中的节点，减小了锁的开销。使用CAS操作支持更高的并发度，在红黑树的创建过程会用synchronized锁定桶头节点。

区别：
* HashMap不是线程安全的，多个线程同时写入的话可能导致数据不一致或抛出异常；而ConcurrentHashMap是线程安全的；
* HashMap在单线程环境中性能比较高，因为没有锁的开销；ConcurrentHashMap在多线程环境中性能更高，因为它通过CAS操作减少了锁的粒度；
* 底层实现上，两个都是使用数组+链表或红黑树，但是ConcurrentHashMap增加了分段锁或CAS操作；
* 扩容的时候ConcurrentHashMap扩容时支持并发操作，可以同时对多个段进行扩容；
* HashMap在迭代过程中如果有修改会抛出ConcurrentModificationException异常；

> 总结的话就是，如果需要在多线程环境中使用线程安全的哈希表，使用ConcurrentHashMap； 如果在单线程环境中，对性能要求较高，使用HashMap

---
### ConcurrentHashMap结构？get时会加锁吗？
get不需要加锁，其他线程的修改会立即可见，因为Entry里的value和next都有volatile修饰，确保可见性。

---
### hashMap为什么不安全？
因为put操作，resize操作都没有加锁，不是原子操作，多线程环境下，可能丢失数据

---
### HashMap有什么问题?头插法，尾插法？
线程不安全

jdk8以后是尾插法，新节点插入链表尾部。避免产生环形链表

---
### HashTable为什么线程安全？
HashTable的方法调用，比如get，put都声明了同步方法，用synchronized修饰。

---
### HashMap和HashSet区别？
HashMap保存键值对，HashSet保存不重复的元素。

---
### 红黑树比链表的优点是什么？
- 查找效率高，时间复杂度O(logn)
- 支持排序

---
### 为什么不一开始就用红黑树？
红黑树每个节点需要保存颜色还有左右子节点指针，插入或删除节点需要旋转来维持平衡。

链表每个节点只需要一个指针，数据量少的时候，链表的初始化和插入效率更高。

---
### 红黑树，平衡二叉树

---
### HashMap扩容条件是什么？为什么数组长度是2的幂次？HashMap的loadFactor值为什么是0.75?
1. 当前元素数量超过阈值，threadhold=capacity * loadFactor
- capacity是数组长度，默认是16，必须是2的幂次
- loadfactor是加载因子，0.75
2. 链表长度达到8，且数组长度小于64，则优先扩容，而非树化。

为什么是2的幂次？
- **计算桶的索引的时候，可以用位运算加速计算**，因为计算索引是index=(n-1)&hash，当n为2的幂次的时候，n-1的二进制全为1，这个时候&运算相当于取模运算，计算效率会更高。

loadFactor为什么是0.75？
- loadFactor值表示哈希表中**存储元素的数量**与**哈希表大小**的比率。当哈希表中存储的元素数量超过了**负载因子乘以哈希表大小**的阈值时，哈希表就需要进行扩容了。
  需要考虑到时间和空间的成本，值太高会增加查询成本，值太低会空间利用率会变低。是一个经验值

为什么链表长度超过8的时候，转化为红黑树？
- 因为按照**泊松分布**，当加载因子为0.75时，长度为8的时候，红黑树查询效率优势明显；

---
### 有没有用过其他框架的HashMap？比如guava模块里的？
1. Guava里用Map创建  Maps.newHashMap();
2. 不可变map：new ImmutableMap.Builder().put().put().build();
3. 有序map：new ImmutableSortedMap.Builder().build()
4. 可以反向把value值映射到key键上，要保证value值唯一：HashBiMap.create()
5. 一个键关联多个值：MultiMap

---
### ConcurrentHashMap扩容时如何解决并发安全问题？在JDK1.8中如何保证线程安全？
用synchronized对桶的头节点加锁，迁移过程中**通过CAS操作更新节点指针**；
