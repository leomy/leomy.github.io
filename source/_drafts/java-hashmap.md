---
title: 浅析Java中的HashMap
toc: true
date: 2018-03-30 13:04:01
categories:
- java
tags: [Algorithm, Java]
---
散列算法是在<strong>时间和空间作出权衡的经典例子，查找、插入的时间复杂度被设计成O(1)。</strong>使用散列的查找算法分为两步，第一步用散列函数将被查找的键转化为数组的一个索引，第二步处理碰撞冲突（拉链法和线性探测法）。

Java中的HashMap采用除数取余法确定索引、拉链法处理冲突。
## 
类的特点（来自源码中对类的描述）：
实现了Map接口，和类HashTable类似支持Map的所有操作，但不同于Hashtable的是HashMap不是同步容器且支持key或value可以为null。HashMap不保证元素间的顺序关系。
当每个元素都通过Hash()正确地分布在桶（bucket）之间，HashMap的基本操作（get/put）在常量时间内完成（时间复杂度为<strong>O(1)</strong>）。对集合视图的迭代所需的时间与HashMap实例的容量（capacity）加上KV键值对的数量（即size()）的结果成正比。因此，当对迭代的性能看重时，初始化容量（initial capacity）不要太高和负载因子（load factor）不要太低。
初始化容量和负载因子，这两个参数会影响HashMap的性能。容量就是HashMap中桶的数量，初始化容量就是创建HashMap实例时的容量。负载因子用来表示最大的真实容量（真正的KV键值对数量）除以容量的结果。当HashMap中元素（entry即对KV对）的数量超过负载银子和当前容量的乘积时，将会重新进行rehash（重建内部数据结构），因此HashMap约有两倍桶的数量。
对于时间和空间成本的考量，通常情况下负载因子为0.75是一个很好的折衷方案。高的负载因子会减少空间消耗但增加查找成本（反应在HashMap的许多操作中，包括get/put）。在设置初始化容量时，应该考虑预期的项数和负载因子，以便减少rehash()的操作。当初始化容量大于最大容量除以负载因子时，不会进行rehash()操作。
如果HashMap的实例要存储许多键值对，更有效率的方式是在创建实例就分配足够大的空间，而不是让它慢慢的进行rehash()到所需的容量。
HashMap并不是同步容器。当多个线程并发访问一个HashMap实例并至少有一个线程会修改实例的结构时，必须在外部使用同步来保证线程安全（添加、删除键值对都会改变结构，修改实例中已存在键值对的值时不是结构改变）。通常通过同步HashMap内某个对象来完成。

## UML
在Oracle发布的jdk7与jdk8对HashMap做出了一些改变
类继承体系
继承关系
![](/images/java-HashMap-hierarchy.jpg)
属性（全部为私有属性）
jdk7的属性![](/images/java-HashMap-properties.jpg)
方法（只列出了公共方法）
jdk7![](/images/java-HashMap-publicMethos.jpg)
构造器
![](/images/java-HashMap-constructors.jpg)
内部类
jdk7![](/images/java-HashMap-innerClasses.jpg)