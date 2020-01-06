---
layout: post
title: "虚拟机运行时数据区域"
subtitle: ' JVM 相关知识 '
author: "Song"
header-style: text
tags:
  - java
  - jvm
---

## 运行时数据区域的划分

划分的区域有各自的用途，以及创建及销毁的时间。有些伴随虚拟机进程启动而一直存在，有些区域则依赖用户线程的启动和结束而建立和销毁。Java 虚拟机规范( SE7 )规定虚拟机运行时数据区域划分为，`程序计数器`，`虚拟机栈`，`native method 栈`，`方法区`，`堆`。

<img src="https://song-dev.github.io/img/in-post/post-git/jvm-runtime-data-area" alt="jvm-runtime-data-area" style="zoom:60%;" />

## 程序计数器

- 程序计数器是线程私有
- 程序计数器为线程行号指示器，处理器一个核心唯一时刻只能执行一条指令，故需要程序计数器记录线程执行指令位置，为了恢复线程继续执行。
- 若执行 java 方法，程序计数器记录字节码指令地址。若执行 native 方法，程序计数器值为空
- 程序计数器是所有区域唯一没有任何 OOM 的区域
- 程序计数器的值为 当前线程内存空间的起始地址(基地址) + 记录字节码指令相对基地址偏移地址

<img src="https://song-dev.github.io/img/in-post/post-git/jvm-runtime-program-counter-mind.png" alt="jvm-runtime-program-counter-mind" style="zoom:60%;" />

## 虚拟机栈

- 虚拟机栈是线程私有的，其生命周期和程序计数器一样，都和线程生命周期相同
- 虚拟机栈描述是 java 方法执行的内存模型，存储`局部变量表`，`操作数栈`，`动态链接`，`方法出口`等信息。

<img src="https://song-dev.github.io/img/in-post/post-git/jvm-program-counter.png" alt="jvm-program-counter" style="zoom:60%;" />

- 每个方法从调用到执行完成的过程，都对应着一个栈帧从虚拟机中入栈到出栈的过程
- 局部变量表所需内存空间在编译期间完成分配，方法运行期间不会改变局部变量表大小。(静态语言特性)
- 当线程请求栈深度大于虚拟机所允许的深度，抛出 StackOverflowError 异常；如果虚拟机栈可以动态扩展，如果扩展时无法申请到足够内存，抛出 OutOfMemoryError 异常

> 局部数组引用存放在栈，数组对象存放在堆中

<img src="https://song-dev.github.io/img/in-post/post-git/jvm-runtime-stack-area.png" alt="jvm-runtime-stack-area" style="zoom:60%;" />

### 本地方法栈

- 本地方法栈为 Native 方法服务
- Hotspot 虚拟机将本地方法栈和虚拟机栈合并
- 同虚拟机栈，存在 StackOverflowError 和 OutOfMemoryError 异常

## java 堆

- 堆是线程共享的
- 几乎所有的对象都在堆上分配，除了栈上分配、标量替换的 `JIT` 编译优化存在的特例
- 垃圾收集器主要管理的区域为堆，现在 GC 基本都采用分代收集算法，可以分为新生代，老年代。或者详细分为 Eden 控件、From Survivor 空间、To Survivor 空间等
- Hotspot 虚拟机中在堆中分配了线程私有的缓冲区，方便快速分配内存，快速回收内存
- 如果堆中没有内存完成实例的分配，且堆无法扩展时，会抛出 OutOfMemoryError 异常

<img src="https://song-dev.github.io/img/in-post/post-git/jvm-runtime-heap-area.png" alt="jvm-runtime-heap-area" style="zoom:60%;" />

## 方法区

- 方法区是线程共享的
- 方法区存储被虚拟机加载的`类信息`、`常量`、`静态变量`、`即时编译器编译后的代码数据`
- 全局变量存放在堆中，其非静态变量，不会独立类的实例存在。类实例化后存放在堆中，即对象存放在堆中，也包括对象的全局变量
- Hotspot 虚拟机将 GC 分代收集算法扩展到方法区，方法区可以称为永久代(和堆永久代不等价)
- 很多其他虚拟机不存在永久代，如 IBM J9、BEA JRockit。Hotspot 在 jdk 1.8 版本已经删除永久代，改用 Native Memory(直接内存，metaspace 元空间) 代替。jdk1.7 已经将字符串常量池从永久代中移除
- GC 在方法区很少出现，主要是针对常量池的回收和对类的卸载(条件苛刻)
- 方法区和堆一样不需要连续的内存和可以选择固定大小，或者可扩展
- 当方法区无法满足内存分配需求时，将抛出 OutOfMemoryError 异常

> 虚拟机加载类信息包括类版本、字段、接口、方法等描述信息

<img src="https://song-dev.github.io/img/in-post/post-git/jvm-runtime-method-area.png" alt="jvm-runtime-method-area" style="zoom:60%;" />

### 运行时数据常量池

- 运行时常量池是方法区一部分，其包括编译期生成各种字面量和符号引用(类、方法、接口等签名信息)
- 字符串常量池在 Hotspot jdk1.7 版本中的永久代移除，字符串常量池属于运行时常量池
- 运行时常量池具备动态性，并不要求编译期才产生，运行期间也可能将新的常量放入常量池中
- 常量池属于方法区一部分，同样会受到方法区内存限制，当常量池中无法申请到内存抛出 OutOfMemoryError 异常

<img src="https://song-dev.github.io/img/in-post/post-git/jvm-runtime-constant-pool.png" alt="jvm-runtime-constant-pool" style="zoom:60%;" />

## 直接内存

定义：NIO类（JDK1.4引入）中基于通道和缓冲区的I/O方式 通过使用Native函数库 直接分配 的堆外内存

- 特点：本机内存不会受到java 堆大小限制，但是会受物理内存和系统内存限制

> 不属于虚拟机运行时数据区的一部分, 也不是虚拟机规范中定义的内存区域, 不在堆中分配

- 应用场景：适用于频繁调用的场景

> 通过一个 存储在Java堆中的DirectByteBuffer对象 作为这块内存的引用 进行操作，从而避免在 Java 堆和 Native堆之间来回复制数据，提高使用性能

- 抛出的异常：OutOfMemoryError，即与其他内存区域的总和 大于 物理内存限制

## 总结

<img src="https://song-dev.github.io/img/in-post/post-git/jvm-runtime-data-area-summarize.png" alt="jvm-runtime-data-area-summarize" style="zoom:60%;" />