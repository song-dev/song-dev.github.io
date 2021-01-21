---
layout: post
title: "ConcurrentHashMap 详细分析"
subtitle: '从源码了解 ConcurrentHashMap 在 JDK 1.7 和 1.8 改变'
author: "Song"
header-style: text
tags:
  - java
  - collection
  - ConcurrentHashMap
---

ConcurrentHashMap 是 Java 并发包中提供的一个线程安全且高效的 HashMap 实现，以弥补 HashMap 不适合在并发环境中操作使用的不足，本文就来分析下 ConcurrentHashMap 的实现原理，并对其实现原理进行分析！

### 一、摘要

在之前的集合文章中，我们了解到 HashMap 在多线程环境下操作可能会导致程序死循环的线上故障！

既然在多线程环境下不能使用 HashMap，那如果我们想在多线程环境下操作 map，该怎么操作呢？

想必阅读过小编之前写的《HashMap 在多线程环境下操作可能会导致程序死循环》一文的朋友们一定知道，其中有一个解决办法就是使用 java 并发包下的 ConcurrentHashMap 类！

今天呢，我们就一起来聊聊 ConcurrentHashMap 这个类！

### 二、简介

众所周知，在 Java 中，HashMap 是非线程安全的，如果想在多线程下安全的操作 map，主要有以下解决方法：

- 第一种方法，使用`Hashtable`线程安全类；
- 第二种方法，使用`Collections.synchronizedMap`方法，对方法进行加同步锁；
- 第三种方法，使用并发包中的`ConcurrentHashMap`类；

在之前的文章中，关于 Hashtable 类，我们也有所介绍，Hashtable 是一个线程安全的类，Hashtable 几乎所有的添加、删除、查询方法都加了`synchronized`同步锁！

相当于给整个哈希表加了一把大锁，多线程访问时候，只要有一个线程访问或操作该对象，那其他线程只能阻塞等待需要的锁被释放，在竞争激烈的多线程场景中性能就会非常差，**所以 Hashtable 不推荐使用！**

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpEib75W5nSUTK4TzvrgoDyN6wiaKP0RB8t1MDvibnz1KnvEFOiaBoibxAa0Jw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

再来看看第二种方法，使用`Collections.synchronizedMap`方法，我们打开 JDK 源码，部分内容如下：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpEQnMGSib749eXKYZVp3T2y8lgGsBjaicGLPO1k9w6AClNOI3TNtJc6fPw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

可以很清晰的看到，如果传入的是 HashMap 对象，其实也是对 HashMap 做的方法做了一层包装，里面使用对象锁来保证多线程场景下，操作安全，本质也是对 HashMap 进行全表锁！

**使用`Collections.synchronizedMap`方法，在竞争激烈的多线程环境下性能依然也非常差，所以不推荐使用！**

上面 2 种方法，由于都是对方法进行全表锁，所以在多线程环境下容易造成性能差的问题，因为** hashMap 是数组 + 链表的数据结构，如果我们把数组进行分割多段，对每一段分别设计一把同步锁，这样在多线程访问不同段的数据时，就不会存在锁竞争了，这样是不是可以有效的提高性能？**

再来看看第三种方法，使用并发包中的`ConcurrentHashMap`类！

**ConcurrentHashMap 类所采用的正是分段锁的思想，将 HashMap 进行切割，把 HashMap 中的哈希数组切分成小数组，每个小数组有 n 个 HashEntry 组成，其中小数组继承自`ReentrantLock（可重入锁）`，这个小数组名叫`Segment`，** 如下图：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpEgEwHHCCJB23y94E0PS2wAoz4BaIeXlfJxeU8Ie5xkAnuDJ66Z6KvWQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

当然，JDK1.7 和 JDK1.8 对 ConcurrentHashMap 的实现有很大的不同！

JDK1.8 对 HashMap 做了改造，**当冲突链表长度大于 8 时，会将链表转变成红黑树结构**，上图是 ConcurrentHashMap 的整体结构，参考 JDK1.7！

我们再来看看 JDK1.8 中 ConcurrentHashMap 的整体结构，内容如下：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpEx9F8m1IeGibbS40iaWkxkibdeYyt3CStlkPq7FVQY4bzt7MfpqCHTBlKA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

JDK1.8 中 ConcurrentHashMap 类取消了 Segment 分段锁，采用 `CAS` + `synchronized` 来保证并发安全，数据结构跟 jdk1.8 中 HashMap 结构类似，都是**数组 + 链表（当链表长度大于 8 时，链表结构转为红黑二叉树**）结构。

ConcurrentHashMap 中 synchronized 只锁定当前链表或红黑二叉树的首节点，只要节点 hash 不冲突，就不会产生并发，相比 JDK1.7 的 ConcurrentHashMap 效率又提升了 N 倍！

说了这么多，我们再一起来看看 ConcurrentHashMap 的源码实现。

### 三、JDK1.7 中的 ConcurrentHashMap

JDK 1.7 的 ConcurrentHashMap 采用了非常精妙的**分段锁**策略，打开源码，可以看到 ConcurrentHashMap 的主存是一个 Segment 数组。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpEA68Utz4EYiadib62AVtIsqDo3z7C7ONFFWibQ7UJXVQN0vURHdic6skyRA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们再来看看 Segment 这个类，在 ConcurrentHashMap 中它是一个静态内部类，内部结构跟 HashMap 差不多，源码如下：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpEp2ia3IL3QC7C4FFakqmYmHnjrqSKDuuFGVNiaB7duRZicZVqf1gINL4XA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

存放元素的 HashEntry，也是一个静态内部类，源码如下：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpEWkibziblYPesBiaazibucia51DDuvica5vdXtGlJMpV9TfR1REsicS3wevEtQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

`HashEntry`和`HashMap`中的 `Entry`非常类似，唯一的区别就是其中的核心数据如`value` ，以及`next`都使用了`volatile`关键字修饰，保证了多线程环境下数据获取时的**可见性**！

从类的定义上可以看到，Segment 这个静态内部类继承了`ReentrantLock`类，`ReentrantLock`是一个可重入锁，如果了解过多线程的朋友们，对它一定不陌生。

`ReentrantLock`和`synchronized`都可以实现对线程进行加锁，不同点是：`ReentrantLock`可以指定锁是公平锁还是非公平锁，操作上也更加灵活，关于此类，具体在以后的多线程篇幅中会单独介绍。

因为`ConcurrentHashMap`的大体存储结构和`HashMap`类似，所以就不对每个方法进行单独分析介绍了，关于`HashMap`的分析，有兴趣的朋友可以参阅小编之前写的《深入分析 HashMap》一文。

ConcurrentHashMap 在存储方面是一个 Segment 数组，一个 Segment 就是一个子哈希表，Segment 里维护了一个 HashEntry 数组，其中 Segment 继承自 ReentrantLock，并发环境下，对于不同的 Segment 数据进行操作是不用考虑锁竞争的，因此不会像 Hashtable 那样不管是添加、删除、查询操作都需要同步处理。

**理论上 ConcurrentHashMap 支持 concurrentLevel（通过 Segment 数组长度计算得来） 个线程并发操作，每当一个线程独占一把锁访问 Segment 时，不会影响到其他的 Segment 操作，效率大大提升！**

上面介绍完了对象属性，我们继续来看看 ConcurrentHashMap 的构造方法，源码如下：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpEektn6tenw8rDG8aSEzY4nfhnPXhcwTHSI5uibDvicRlCYVGVu838TXkw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

`this`调用对应的构造方法，源码如下：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpE0PvljjGLXFZW6PgObADEJgp5w5YuuBVGic36T1vLcxOwF3icvGzPEvOg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

从源码上可以看出，ConcurrentHashMap 初始化方法有三个参数，**initialCapacity（初始化容量）为 16、loadFactor（负载因子）为 0.75、concurrentLevel（并发等级）为 16**，如果不指定则会使用默认值。

其中，**值得注意的是 concurrentLevel 这个参数，虽然 Segment 数组大小 ssize 是由 concurrentLevel 来决定的，但是却不一定等于 concurrentLevel，ssize 通过位移动运算，一定是大于或者等于 concurrentLevel 的最小的 2 的次幂！**

通过计算可以看出，按默认的 initialCapacity 初始容量为 16，concurrentLevel 并发等级为 16，理论上就允许 16 个线程并发执行，并且每一个线程独占一把锁访问 Segment，不影响其它的 Segment 操作！

从之前的文章中，我们了解到 HashMap 在多线程环境下操作可能会导致程序死循环，仔细想想你会发现，造成这个问题无非是 put 和扩容阶段发生的！

那么这样我们就可以从 put 方法下手了，来看看 ConcurrentHashMap 是怎么操作的？

#### 3.1、put 操作

ConcurrentHashMap 的 put 方法，源码如下：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpElIqnFM9lUFk3ARzT5eP4yPNVD3nQLpfqK2tq3bjFTmnv2BBmlsWhBg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

从源码可以看出，这部分的 put 操作主要分两步：

- 定位 Segment 并确保定位的 Segment 已初始化；
- 调用 Segment 的 put 方法；

真正插入元素的 put 方法，源码如下：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpEnYf1OBZ9BSzd1XpI95vK6o5R8rX4hXBE5fKE0rgw5ny8pOcfIpMMiaQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

从源码可以看出，真正的 put 操作主要分以下几步：

- 第一步，尝试获取对象锁，如果获取到返回 true，否则执行`scanAndLockForPut`方法，这个方法也是尝试获取对象锁；
- 第二步，获取到锁之后，类似 hashMap 的 put 方法，通过 key 计算所在 HashEntry 数组的下标；
- 第三步，获取到数组下标之后遍历链表内容，通过 key 和 hash 值判断是否 key 已存在，如果已经存在，通过标识符判断是否覆盖，默认覆盖；
- 第四步，如果不存在，采用头插法插入到 HashEntry 对象中；
- 第五步，最后操作完整之后，释放对象锁；

我们再来看看，上面提到的`scanAndLockForPut`这个方法，源码如下：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpEGM5FchtjRT33MXsWHFibiaSPdZDRsZkJsYVT435eFibFtiaG07es6qhicLg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

`scanAndLockForPut`这个方法，操作也是分以下几步：

- 当前线程尝试去获得锁，查找 key 是否已经存在，如果不存在，就创建一个 HashEntry 对象；
- 如果重试次数大于最大次数，就调用`lock()`方法获取对象锁，如果依然没有获取到，当前线程就阻塞，直到获取之后退出循环；
- 在这个过程中，key 可能被别的线程给插入，所以在第 5 步中，如果 HashEntry 存储内容发生变化，重置重试次数；

通过`scanAndLockForPut()`方法，当前线程就可以在即使获取不到`segment`锁的情况下，完成需要添加节点的实例化工作，当获取锁后，就可以直接将该节点插入链表即可。

这个方法还实现了**类似于自旋锁的功能，循环式的判断对象锁是否能够被成功获取，直到获取到锁才会退出循环，防止执行 put 操作的线程频繁阻塞，这些优化都提升了 put 操作的性能。**

#### 3.2、get 操作

get 方法就比较简单了，因为不涉及增、删、改操作，所以不存在并发故障问题，源码如下：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpEFibdSfrQleewx5ciaVDt3iaEd54DuiaeTA90VFgIe3KVLn5iaFEFnhot8yg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

由于 HashEntry 涉及到的共享变量都使用 volatile 修饰，volatile 可以保证内存可见性，所以不会读取到过期数据。

#### 3.3、remove 操作

remove 操作和 put 方法差不多，都需要获取对象锁才能操作，通过 key 找到元素所在的 Segment 对象然后移除，源码如下：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpEdSgXedGPGIIhIByoqw85dh9G5hUAv6Oia3xow3Ic6h49iaLqxOO1yeGA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

与 get 方法类似，先获取 Segment 数组所在的 Segment 对象，然后通过 Segment 对象去移除元素，源码如下：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpEbToa3lIicnwcTYSa8Zdvlr0d1lwMibzJlbBibT4dy4iaDIUF4IQ2MdjYSQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

先获取对象锁，如果获取到之后执行移除操作，之后的操作类似 hashMap 的移除方法，步骤如下：

- 先获取对象锁；
- 计算 key 的 hash 值在 HashEntry[]中的角标；
- 根据 index 角标获取 HashEntry 对象；
- 循环遍历 HashEntry 对象，HashEntry 为单向链表结构；
- 通过 key 和 hash 判断 key 是否存在，如果存在，就移除元素，并将需要移除的元素节点的下一个，向上移；
- 最后就是释放对象锁，以便其他线程使用；

### 四、JDK1.8 中的 ConcurrentHashMap

虽然 JDK1.7 中的 ConcurrentHashMap 解决了 HashMap 并发的安全性，但是当冲突的链表过长时，在查询遍历的时候依然很慢！

在 JDK1.8 中，HashMap 引入了红黑二叉树设计，当冲突的链表长度大于 8 时，会将链表转化成红黑二叉树结构，红黑二叉树又被称为平衡二叉树，在查询效率方面，又大大的提高了不少。

![图片](https://mmbiz.qpic.cn/mmbiz_png/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpETbpOhjWeMibxDibeVaAPjSDjLq6f7TPu0T7zsZz8p0WbJQTdVO47j2uw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

因为 HashMap 并不支持在多线程环境下使用， JDK1.8 中的 ConcurrentHashMap 和往期 JDK 中的 ConcurrentHashMa 一样支持并发操作，整体结构和 JDK1.8 中的 HashMap 类似，相比 JDK1.7 中的 ConcurrentHashMap， 它抛弃了原有的 Segment 分段锁实现，采用了 `CAS + synchronized` 来保证并发的安全性。

JDK1.8 中的 ConcurrentHashMap 对节点`Node`类中的共享变量，和 JDK1.7 一样，使用`volatile`关键字，保证多线程操作时，变量的可见行！

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpEmearOb7v6lrcyjvEO95VfDgQkLpWn8zazLVfbZQx7TsxiaaXdQZ37Lg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

其他的细节，与 JDK1.8 中的 HashMap 类似，我们来具体看看 put 方法！

#### 4.1、put 操作

打开 JDK1.8 中的 ConcurrentHashMap 中的 put 方法，源码如下：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpERhkOiaVIPKwrkToht4DxyLEmayMhclYmjeB9uxIxy7vMu3NJ6kJRQhA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

当进行 put 操作时，流程大概可以分如下几个步骤：

- 首先会判断 key、value 是否为空，如果为空就抛异常！
- 接着会判断容器数组是否为空，如果为空就初始化数组；
- 进一步判断，要插入的元素`f`，在当前数组下标是否第一次插入，如果是就通过 CAS 方式插入；
- 在接着判断`f.hash == -1`是否成立，如果成立，说明当前`f`是`ForwardingNode`节点，表示有其它线程正在扩容，则一起进行扩容操作；
- 其他的情况，就是把新的`Node`节点按链表或红黑树的方式插入到合适的位置；
- 节点插入完成之后，接着判断链表长度是否超过`8`，如果超过`8`个，就将链表转化为红黑树结构；
- 最后，插入完成之后，进行扩容判断；

put 操作大致的流程，就是这样的，可以看的出，复杂程度比 JDK1.7 上了一个台阶。

##### 4.1.1、initTable 初始化数组

我们再来看看源码中的第 3 步 `initTable()`方法，如果数组为空就**初始化数组**，源码如下：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpEebjU0tyGtks1AfLYl5RqBMFJ7ibPYavq5vAHQVsOtJTxlia4pP8y7icmQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

sizeCtl 是一个对象属性，使用了 volatile 关键字修饰保证并发的可见性，默认为 0，当第一次执行 put 操作时，通过`Unsafe.compareAndSwapInt()`方法，俗称`CAS`，将 `sizeCtl`修改为 `-1`，有且只有一个线程能够修改成功，接着执行 table 初始化任务。

如果别的线程发现`sizeCtl<0`，意味着有另外的线程执行 CAS 操作成功，当前线程通过执行`Thread.yield()`让出 CPU 时间片等待 table 初始化完成。

##### 4.1.2、helpTransfer 帮组扩容

我们继续来看看 put 方法中第 5 步`helpTransfer()`方法，如果`f.hash == -1`成立，说明当前`f`是`ForwardingNode`节点，意味有其它线程正在扩容，则一起进行扩容操作，源码如下：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpEsCreQkwCLHVmGfEkia3mDMIQvx9NT7FQPRYibTzzcXDrQca3OXAIwANA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这个过程，操作步骤如下：

- 第 1 步，对 table、node 节点、node 节点的 nextTable，进行数据校验；
- 第 2 步，根据数组的 length 得到一个标识符号；
- 第 3 步，进一步校验 nextTab、tab、sizeCtl 值，如果 nextTab 没有被并发修改并且 tab 也没有被并发修改，同时 `sizeCtl < 0`，说明还在扩容；
- 第 4 步，对 sizeCtl 参数值进行分析判断，如果不满足任何一个判断，将`sizeCtl + 1`, 增加了一个线程帮助其扩容;

##### 4.1.3、addCount 扩容判断

我们再来看看源码中的第 9 步 `addCount()`方法，插入完成之后，扩容判断，源码如下：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpEXnJHlcDeEjr057xarZBBGj8Zp6MWooWI5GKxLZsOtIQKaGeRlqVNOQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这个过程，操作步骤如下：

- 第 1 步，利用 CAS 将方法更新 baseCount 的值
- 第 2 步，检查是否需要扩容，默认 check = 1，需要检查；
- 第 3 步，如果满足扩容条件，判断当前是否正在扩容，如果是正在扩容就一起扩容；
- 第 4 步，如果不在扩容，将 sizeCtl 更新为负数，并进行扩容处理；

put 的流程基本分析完了，可以从中发现，里面大量的使用了`CAS`方法，CAS 表示比较与替换，里面有 3 个参数，分别是**目标内存地址、旧值、新值**，每次判断的时候，会将旧值与目标内存地址中的值进行比较，如果相等，就将新值更新到内存地址里，如果不相等，就继续循环，直到操作成功为止！

虽然使用的了`CAS`这种乐观锁方法，但是里面的细节设计的很复杂，阅读比较费神，有兴趣的朋友们可以自己研究一下。

#### 4.2、get 操作

get 方法操作就比较简单了，因为不涉及并发操作，直接查询就可以了，源码如下：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpEneUWWpojMqF84RrmajFpiaCgAlauib5X4oQ4qYDM5qTicbibRvI3A1696g/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

从源码中可以看出，步骤如下：

- 第 1 步，判断数组是否为空，通过 key 定位到数组下标是否为空；
- 第 2 步，判断 node 节点第一个元素是不是要找到，如果是直接返回；
- 第 3 步，如果是红黑树结构，就从红黑树里面查询；
- 第 4 步，如果是链表结构，循环遍历判断；

#### 4.3、reomve 操作

reomve 方法操作和 put 类似，只是方向是反的，源码如下：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpEInDU9ftF70ndMPfCpgOeTDiaK0RVnyMdtkXSWXt48YvDAcRlXCqDOSg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

replaceNode 方法，源码如下：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw4Wic7A4mAo0R6EicrDt7sSpESGcneibT7HjcsnNVJJRvdnjd1JGAibX2L6vPatLGUbgE5ceKFyNg8Fbw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

从源码中可以看出，步骤如下：

- 第 1 步，循环遍历数组，接着校验参数；
- 第 2 步，判断是否有别的线程正在扩容，如果是一起扩容；
- 第 3 步，用 synchronized 同步锁，保证并发时元素移除安全；
- 第 4 步，因为 `check= -1`，所以不会进行扩容操作，利用 CAS 操作修改 baseCount 值；

### 五、总结

虽然 HashMap 在多线程环境下操作不安全，但是在 `java.util.concurrent` 包下，java 为我们提供了 ConcurrentHashMap 类，保证在多线程下 HashMap 操作安全！

在 JDK1.7 中，ConcurrentHashMap 采用了分段锁策略，将一个 HashMap 切割成 Segment 数组，其中 Segment 可以看成一个 HashMap， 不同点是 Segment 继承自 ReentrantLock，在操作的时候给 Segment 赋予了一个对象锁，从而保证多线程环境下并发操作安全。

但是 JDK1.7 中，HashMap 容易因为冲突链表过长，造成查询效率低，所以在 JDK1.8 中，HashMap 引入了红黑树特性，当冲突链表长度大于 8 时，会将链表转化成红黑二叉树结构。

在 JDK1.8 中，与此对应的 ConcurrentHashMap 也是采用了与 HashMap 类似的存储结构，但是 JDK1.8 中 ConcurrentHashMap 并没有采用分段锁的策略，而是在元素的节点上采用 `CAS + synchronized` 操作来保证并发的安全性，源码的实现比 JDK1.7 要复杂的多。

本文因为是断断续续写出来，如果有理解不对的地方，欢迎各位网友指出！

### 六、参考

1、JDK1.7&JDK1.8 源码

2、**JavaGuide - 容器 - ConcurrentHashMap**[1]

3、**博客园 -dreamcatcher-cx - ConcurrentHashMap1.7 实现原理及源码分析**[2]

4、**简书 -占小狼 - 深入浅出 ConcurrentHashMap1.8**[3]

> 转载 https://mp.weixin.qq.com/s/B1XboYOpGfnIOtCUvmOSmA