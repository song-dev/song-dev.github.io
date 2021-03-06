---
layout: post
title: "虚拟机中对象"
subtitle: ' JVM 相关知识 '
author: "Song"
header-style: text
tags:
  - java
  - JVM
  - object
---

## 1. 对象创建

- 对象创建需要 new 关键字

> 数组和类的class对象除外

### 1.1 创建过程

当遇到关键字new指令时，Java对象创建过程便开始，整个过程如下：

<img src="https://song-dev.github.io/img/in-post/post-jvm/jvm-object-create-process.png" alt="jvm-object-create-process" style="zoom:80%;" />

### 1.2 过程步骤

#### 步骤1：类加载检查

1. 检查 该new指令的参数 是否能在 常量池中 定位到一个类的符号引用
2. 检查 该类符号引用 代表的类是否已被加载、解析和初始化过

> 如果没有，需要先执行相应的类加载过程

#### 步骤2：为对象分配内存

- 虚拟机将为对象分配内存，即把一块确定大小的内存从 Java 堆中划分出来

> 虚拟机加载 class 之后，对象所需内存大小可完全确定

- 关于分配内存，此处主要讲解内存分配方式
- 内存分配 根据 Java堆内存是否绝对规整 分为两种方式：指针碰撞 & 空闲列表

> Java堆内存 规整：已使用的内存在一边，未使用内存在另一边
> Java堆内存 不规整：已使用的内存和未使用内存相互交错

<img src="https://song-dev.github.io/img/in-post/post-jvm/jvm-heap-memory.png" alt="jvm-heap-memory" style="zoom:80%;" />

##### 方式1：指针碰撞

- 假设Java堆内存绝对规整，内存分配将采用指针碰撞
- 分配形式：已使用内存在一边，未使用内存在另一边，中间放一个作为分界点的指示器

<img src="https://song-dev.github.io/img/in-post/post-jvm/jvm-指针碰撞-正常.png" alt="jvm-指针碰撞-正常" style="zoom:80%;" />

- 那么，分配对象内存 = 把指针向 未使用内存 移动一段 与对象大小相等的距离

<img src="https://song-dev.github.io/img/in-post/post-jvm/jvm-指针碰撞-分配内存空间.png" alt="jvm-指针碰撞-分配内存空间" style="zoom:70%;" />

##### 方式2：空闲列表

- 假设Java堆内存不规整，内存分配将采用 空闲列表
- 分配形式：虚拟机维护着一个 记录可用内存块 的列表，在分配时从列表中找到一块足够大的空间划分给对象实例，并更新列表上的记录

#### 额外知识

- 分配方式的选择 取决于 Java堆内存是否规整；
- 而 Java堆是否规整 由所采用的垃圾收集器是否带有压缩整理功能决定。因此：

1. 使用带 Compact 过程的垃圾收集器时，采用指针碰撞；

> 如Serial、ParNew垃圾收集器

2. 使用基于 Mark_sweep算法的垃圾收集器时，采用空闲列表。

> 如 CMS垃圾收集器

#### 特别注意

- 对象创建在虚拟机中是非常频繁的操作，即使仅仅修改一个指针所指向的位置，在并发情况下也会引起线程不安全

> 如，正在给对象A分配内存，指针还没有来得及修改，对象B又同时使用了原来的指针来分配内存

**所以，给对象分配内存会存在线程不安全的问题。**

解决 线程不安全 有两种方案：

1. 同步处理分配内存空间的行为

> 虚拟机采用 CAS + 失败重试的方式 保证更新操作的原子性

2. 把内存分配的动作按照线程划分在不同的空间之中进行

> 1. 即每个线程在 Java堆中预先分配一小块内存（本地线程分配缓冲（Thread Local Allocation Buffer ，TLAB）），哪个线程要分配内存，就在哪个线程的TLAB上分配，只有TLAB用完并分配新的TLAB时才需要同步锁。
> 2. 虚拟机是否使用TLAB，可以通过-XX:+/-UseTLAB参数来设定。

#### 步骤3： 将内存空间初始化为零值

内存分配完成后，虚拟机需要将分配到的内存空间初始化为零（不包括对象头）

> 1. 保证了对象的实例字段在使用时可不赋初始值就直接使用（对应值 = 0）
> 2. 如使用本地线程分配缓冲（TLAB），这一工作过程也可以提前至TLAB分配时进行(即给线程分配独立内存空间时已经初始化内存为零值)。

#### 步骤4： 对对象进行必要的设置

> 如，设置 这个对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的GC分代年龄等信息。

**这些信息存放在对象的对象头中。**

- 至此，从 Java 虚拟机的角度来看，一个新的 Java对象创建完毕
- 但从 Java 程序开发来说，对象创建才刚开始，需要进行一些初始化操作
- 对象头设置后，执行 <init> 方法初始化对象

### 1.3 总结
下面用一张图总结 Java对象创建的过程

<img src="https://song-dev.github.io/img/in-post/post-jvm/jvm-对象分配内存-总结.png" alt="jvm-对象分配内存-总结" style="zoom:90%;" />

## 2. 对象内存布局

Hotspot 虚拟机中对象内存布局分为三部分

- 对象头(Header)
- 实例数据(Instance Data)
- 对齐填充(Padding)

<img src="https://song-dev.github.io/img/in-post/post-jvm/jvm-对象内存布局.png" alt="jvm-对象内存布局" style="zoom:80%;" />

### 2.1 对象头

对象头存储信息包括两个部分(Hotspot)

> 对象头大小为 8 字节的 1 倍或者 2 倍

#### 2.1.1 对象自身的运行时数据(Mark Word)

> 1. 如哈希码，GC 分代年龄，锁标志等
> 2. 第一部分数据称为 Mark Word，其在 32和64位虚拟机中长度分别为 32bit和64bit
> 3. 由于对象需要存储的运行时数据很多，Mark Word 被设计为非固定的数据结构，其根据自身状态复用存储空间
> 4. 在 32bit 虚拟机上，未锁定状态，25bit 存储对象哈希码，4bit 存储对象分代年龄，2bit 存储锁标志位，1bit 固定为 0

存储内容 | 标志位 | 状态
-----    | ------ | ---
对象哈希码、对象分代年龄 | 01 | 未锁定
指向锁记录指针 | 00 | 轻量级锁
指向重量级锁的指针 | 10 | 膨胀（重量级锁定）
空，不需要记录信息 | 11 | GC 标记
偏向线程 ID ，偏向时间戳，对象分代年龄 | 01 | 可偏向

> Hotspot 虚拟机对象头 Mark Word

#### 2.1.2 对象头类型指针
- 对象头另外部分存储类型指针，即对象类元数据的指针，通过指针确定对象是哪个类实例
- 非 Hotspot 虚拟机中对象可能不保存类元数据指针，其通过句柄访问对象

#### 2.1.3 注意

如果对象为数组，则对象头需保存数组长度，这样可以通过对象元数据(类)和数组长度确定对象大小

### 2.2 实例数据

- 实例数据存储父类继承或者本身定义字段
- 存储顺序受虚拟机分配策略参数(FieldAllocationStyle)和字段在 Java 源码中顺序影响
- Hotspot 虚拟机中相同宽度字段被分配到一起
- 父类中定义变量会出现在子类前，不包括相同宽度字段分配一起限制，不包括CompacFields配置为True，子类较窄字段插入到父类变量空隙中

### 2.3 对象对齐填充

虚拟机中对象开始地址必须 8byte 整数倍，且实例数据长度没有对齐定，需要 Padding 对齐补充来补全

> 前提是虚拟机有对齐限制，如 Hotspot 有这限制

### 2.4 总结

<img src="https://song-dev.github.io/img/in-post/post-jvm/jvm-对象内存布局-总结.png" alt="jvm-对象内存布局-总结" style="zoom:100%;" />

## 3. 对象访问定位

虚拟机规范并未规定怎样访问栈上的 reference 对象，只规定了一个指向对象的引用。具体访问方式虚拟机自由实现，目前主流访问方式有：

- 句柄访问
- 直接指针访问

### 3.1 通过句柄访问对象

<img src="https://song-dev.github.io/img/in-post/post-jvm/jvm-通过句柄访问对象.png" alt="jvm-通过句柄访问对象" style="zoom:70%;" />

句柄池最大好处是 reference 中存储的是稳定的句柄地址，在 GC 时只会改变句柄池中实例数据指针，而 reference 本身不修改

### 3.2 通过直接指针对象访问数据

<img src="https://song-dev.github.io/img/in-post/post-jvm/jvm-通过直接指针访问对象.png" alt="jvm-通过直接指针访问对象" style="zoom:80%;" />

直接访问最大好处是速度快，节省一次指针定位时间开销。Hotspot 虚拟机使用直接指针访问对象

### 3.3 说明

<img src="https://song-dev.github.io/img/in-post/post-jvm/jvm-对象访问定位-总结.png" alt="jvm-对象访问定位-总结" style="zoom:100%;" />

## 参考
- [https://www.jianshu.com/p/1952061502d0](https://www.jianshu.com/p/1952061502d0)
