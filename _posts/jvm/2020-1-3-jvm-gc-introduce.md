---
layout: post
title: "垃圾收集器介绍"
subtitle: ' JVM 相关知识 '
author: "Song"
header-style: text
tags:
  - java
  - JVM
  - gc
---

## 垃圾收集器介绍

垃圾收集器主要针对方法区和堆

## 1. 对象死亡判断方式
垃圾收集器对 Java 堆里的对象 是否进行回收的判断准则：Java对象是存活 or 死亡

> 判断对象为死亡才会进行回收

- 在Java虚拟机中，判断对象是否存活有2种方法：

1. 引用计数法
2. 引用链法（可达性分析法）

### 1.1 引用计数法

#### 1.1.1 方式描述

- 给 Java 对象添加一个引用计数器
- 每当有一个地方引用它时，计数器 +1；引用失效则 -1；

#### 1.1.2 判断对象存活准则
当计数器不为 0 时，判断该对象存活；否则判断为死亡（计数器 = 0）。

#### 1.1.3 优点

- 实现简单
- 判断高效

#### 1.1.4 缺点

- 主流 java 虚拟机并未使用引用计数算法管理内存，因为其很难解决对象间相互循环引用问题

> Python语言、游戏脚本等语言使用引用计数算法管理内存

- 具体描述

```
<-- 背景 -->
// 对象objA 和 objB 都有字段 name
// 两个对象相互进行引用，除此之外这两个人对象没有任何引用
objA.name = objB；
objB.name = objA；

<-- 问题 -->
// 实际上这两个对象已经不可能再被访问，应该要被垃圾收集器进行回收
// 但因为他们相互引用，所以导致计数器不为0，这导致引用计数算法无法通知垃圾收集器回收该两个对象
```

**正由于该算法存在判断逻辑漏洞，所以 Java虚拟机没有采用该算法判断Java是否存活。**

### 1.2 引用链法（可达性分析法）

主流商用语言(Java C#)的主流实现中，都是使用可达性分析来判定对象是否存活

1. 可达性分析
2. 第一次标记 & 筛选
3. 第二次标记 & 筛选

#### 1.2.1 可达性分析

##### a. 方式描述

以 GC Roots 对象为起始点，向下搜索，搜索走过路径为引用链，当一个对象到 GC Roots 没有任何引用链相连时，则此对象是不可用的

> - 可作为 GC Root 的对象有:
>   1. Java虚拟机栈（栈帧的本地变量表）中引用的对象
>   2. 本地方法栈中 JNI (Native 方法)引用的对象
>   3. 方法区中类静态属性引用的对象
>   4. 方法区中常量引用的对象
> - 向下搜索的路径 = 引用链

如下图：

<img src="https://song-dev.github.io/img/in-post/post-jvm/jvm-可达性分析示意图.png" alt="jvm-可达性分析示意图" style="zoom:85%;" />

##### b. 判断 对象是否可达 标准

当一个对象到 GC Roots 

> 没有任何引用链相连时，则判断该对象不可达

没有任何引用链相连 = GC Root到对象不可达 = 对象不可用

<img src="https://song-dev.github.io/img/in-post/post-jvm/jvm-可达性分析过程.png" alt="jvm-可达性分析过程" style="zoom:85%;" />

#### 特别注意

- 可达性分析 仅仅只是判断对象是否可达，但还不足以判断对象是否存活 / 死亡
- 当在 可达性分析 中判断不可达的对象，只是“被判刑” = 还没真正死亡
- native 方法需要清除对堆中对象的引用

> 不可达对象会被放在”即将回收“的集合里。

- 要判断一个对象真正死亡，还需要经历两个阶段：
1. 第一次标记 & 筛选
2. 第二次标记 & 筛选

### 1.2.2 第一次标记 & 筛选
- 对象 在 可达性分析中 被判断为不可达后，会被第一次标记 & 准备被筛选

> - 不筛选：继续留在"即将回收"的集合里，等待回收;
> - 筛选：从 ”即将回收“的集合取出

- 筛选的标准：该对象是否有必要执行 finalize()方法

> 1. 若有必要执行（人为设置），则筛选出来，进入下一阶段（第二次标记 & 筛选）；
> 2. 若没必要执行，判断该对象死亡，不筛选 并等待回收

> 当对象无 finalize()方法 或 finalize()已被虚拟机调用过，则视为“没必要执行”

### 1.2.3 第二次标记 & 筛选
当对象经过了第一次的标记 & 筛选，会被进行第二次标记 & 准备被进行 筛选

#### a. 方式描述
该对象会被放到一个 F-Queue 队列中，并由 虚拟机自动建立、优先级低的Finalizer 线程去执行 队列中该对象的finalize()

> 1. finalize()只会被执行一次
> 2. 但并不承诺等待finalize()运行结束。这是为了防止 finalize()执行缓慢 / 停止 使得 F-Queue队列其他对象永久等待。

#### b. 筛选标准
- 在执行finalize()过程中，若对象依然没与引用链上的GC Roots 直接关联 或 间接关联（即关联上与GC Roots 关联的对象），那么该对象将被判断死亡，不筛选（留在”即将回收“集合里） 并 等待回收
- finalize() 方法是对象逃脱最后一次机会,如果在 finalize() 方法中拯救了自己，则当前对象仍然可以存活, 但只有一次机会。(如把 this 复制给类变量，或者对象的成员变量)

### 1.3 总结
3步骤 + 以下流程

<img src="https://song-dev.github.io/img/in-post/post-jvm/jvm-判断对象死亡-总结.png" alt="jvm-判断对象死亡-总结" style="zoom:120%;" />

## 2. 四种引用类型
判断对象是否已死都和引用相关，jdk1.2 之前引用定义很传统：若 reference 类型的数据存储的数值代表另外一块起始地址，称这块内存代表一个引用。局限是无法表示引用的各种状态。

- jdk1.2 之后引用扩充为 强引用、软引用、弱引用、虚引用四种。真四种引用强度依次减弱
- 强引用。大部分引用为强引用，只要强引用存在，被引用的对象永远无法回收
- 软引用。描述一些有用但是并非必需的对象。系统将要发生内存溢出时，会把这些对象列入可回收返回进行二次回收。若还是内存不足，则 OOM 异常。使用 SoftReference 类实现软引用
- 弱引用。描述非必须对象。无论内存是否足够，垃圾收集器工作时，都会回收被弱引用关联的对象。WeakReference 类实现弱引用
- 虚引用。其为最弱一种引用关系。对象是否有虚引用的存在，完全不会影响其生存时间，且无法通过虚引用获得对象实例。为对象设置虚引用目的为在对象被回收时，收到系统通知(监听 GC 工作)。使用 PhantomReference 类实现。

> Android 9 之后，软引用也会在 GC 工作时，其引用对象会直接被回收

## 3. 回收方法区
- java 虚拟机规范不要求在方法区实现垃圾收集，且在方法区垃圾收集效率很低。在堆的新生代中一次 GC 可以回收 70%~95% 的空间，而永久代(方法区，Hotspot 后续版本移除永久代概念)效率远低于新生代。

- 永久代垃圾收集器主要收集两个部分，废弃常量和无用的类
- 废弃常量回收和堆中对象类似，如字符串，若当前系统中没有任何一个 String 对象引用常量池中 "abc" 常量，也没有其他地方引用这个字面量，如果发生 GC ，有必要的话这个常量会别清除出常量池。常量池中的其他类、方法、字段、符号引用也和这字符串常量类似

判定类为 **无用的类** 三个条件
1. 该类所有实例都已被回收，即堆中不存在该类任何实例
2. 加载该类的 ClassLoader 已经被回收
3. 该类对应的 java.lang.Class 对象没有在任何地方被引用，无法再任何地方通过反射访问该类的方法

> 虚拟机对满足上述条件的无用类进行回收，但是受虚拟机 -Xnoclassgc 参数控制。

- 大年使用动态代理、反射框架、动态生成jsp、OSGI 这类频繁自定义 ClassLoader 的场景都需要虚拟机具有卸载类功能，保证永久代不 OOM 异常。

## 4. 垃圾收集算法
见垃圾收集算法详解

## 5. Hotspot 的算法实现

### 5.1 枚举根节点

#### 5.1.1 定义

遍历 GC Roots 节点找引用链操作

> 可作为 GC Roots 节点主要为 常量、静态属性、本地变量表

#### 5.1.2 面临问题

- 很多时候方法区内存就有上百兆，如果检查所有的引用，则会消耗很长时间
- Stop-The—World，GC 进行时候，停顿所以 java 执行线程

> - 因为，枚举根节点必须在一个确保一致性的快照中进行，即整个系统看起来被冻结在某个时间点上，不可以出现分析过程中对象引用关系还在不断变化情况，所以才有 Stop-The—World。
> - 即使在号称几乎不会发生停顿的 CMS 收集器中，枚举根节点时还说会停顿

#### 5.1.3 解决方案

在 Hotspot 虚拟机中，在类加载完成时，Hotspot 就把对象内什么偏移量上什么类型数据计算出来，且在 JIT 编译过程中，也会在特定位置记录栈和寄存器中哪些位置是引用。将这些记录的引用数据，存储在一个 OopMap 数据结构中。

- 虚拟机停顿后，不需要检查所有的执行上下文和全局引用位置
- GC 发生时不需要遍历所有引用，直接分析 OopMap 即可

> 通过上述两种方式解决枚举根节点耗时问题

#### 5.1.4 注意事项

并不是每条指令都存储 OopMap ，否则消耗大量额外存储空间

### 5.2 安全点

#### 5.2.1 背景

Hotspot 可以根据 OopMap 快速完成 GC Roots 枚举，但是 OopMap 内容变化指令非常多，若为每条指令生成对应的 OopMap ，则会消耗大量额外空间，GC 空间成本飙升。

#### 5.2.2 解决方案

- 并未为每条指令生成 OopMap，只在特殊的位置记录，即为安全点(Safepoint)

> 安全点记录 OopMap，以便枚举根节点

- 程序执行时，并非在所有的地方都能停顿下来开始 GC，只有在到达安全点才暂停
- 安全点多少要适中，以程序“是否具有让程序长时间执行的特征”为标准选定

> 长时间运行非指令流太长，而是指令序列复用，如: 方法调用、循环跳转、异常跳转等。具有这些功能的指令才会产生 安全点(Safepoint)

- 在 GC 发生时让所有线程(不包括 GC 执行JNI调用的线程)都运行到最近的安全点上停顿下来有两种方案

#### 5.2.3 抢先试中断
不需要线程执行代码主动配合，在 GC 发生时，首先把所有线程全部中断，如果发现有线程中断的地方不在安全点上，就恢复线程，让他跑到安全点上

> 目前几乎没有虚拟机实现采用抢先式中断来暂停线程，而响应 GC 事件

#### 5.2.4 主动式中断
当 GC 需要中断线程时候，不直接对线程操作，而是设置一个标志，各个线程执行时主动轮询这个标志，发现中断标志为真时中断挂起

> 当需要暂停线程时，就会产生一个自陷异常信号，在预先注册的异常处理器中暂停线程等待，这样一条指令即可完成安全点轮询和出发线程中断

#### 5.2.5 注意事项
轮询标志位置和安全点重合，也与创建对象需要分配内存地方重合。

> - 当线程执行过程中会不断轮询 `轮询标志`，且在轮询标志位置给对象分配内存

### 5.3 安全区域
- Safepoint 机制保证了程序执行时，一定时间内遇到可进入 GC 的 Safepoint，但是程序不执行，如进入等待或者阻塞状态的线程。
- 等待的线程无法响应 JVM 的中断请求，被中断挂起，且无法等待线程重新被分配 CPU 时间，这时使用安全区域解决

#### 5.3.1 定义
一段代码片段中，引用关系不会发生变化。在这个区域中任意地方开始 GC 都是安全的，可以把 Safe Region 看做是被扩展的 Safepoint

#### 5.3.2 实现步骤
1. 线程执行到 Safe Region 中的代码时，标识已经进入 Safe Region
2. 进入 Safe Region 后，JVM 要发起 GC 时候，不用管标识自己为 Safe Region 状态线程
3. 线程离开 Safe Region 时，要检查系统时候已经完成了根节点枚举(或者 GC 过程)，如果完成了线程继续执行，否则必须等到直到收到可以离开 Safe Region 的信号为止

## 6. 理解 GC 日志

## 7. 垃圾收集器参数总结

