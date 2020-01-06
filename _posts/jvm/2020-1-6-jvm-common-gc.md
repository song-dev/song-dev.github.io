---
layout: post
title: "常见垃圾收集器"
subtitle: ' JVM 相关知识 '
author: "Song"
header-style: text
tags:
  - java
  - JVM
  - gc
---


## 垃圾收集器类型

- 垃圾收集器是垃圾收集算法的具体实现
- 现在主流的垃圾收集器有 7 种
- 我们会根据需求场景的不同，选择不同特点的垃圾收集器

<img src="https://upload-images.jianshu.io/upload_images/944365-4ff2b4ddab3474e0.png" alt="垃圾收集器类别.png" style="zoom:40%;" />

## 1. Serial收集器

### 1.1 定义

最基本、发展历史最长的垃圾收集器

### 1.2 优点

- 并发收集

在进行垃圾收集时，必须暂停其他所有工作线程（`Stop The World`），直到收集结束。

> 暂停工作线程 是在用户不可见的情况下进行

> 注：并发 与 并行的区别 <br>
a. 并发：在 某一时段内，交替执行多个任务（即先处理A再处理B，循环该过程）<br>
b. 并行：在 某一时刻内，同时执行多个任务（即同时处理A、B）

- 单线程

只使用 一条线程 完成垃圾收集（`GC` 线程）

- 效率高

对于限定单`CPU`环境来说，`Serial`收集器没有线程交互开销（专一做垃圾收集），拥有更高的单线程收集效率。

> 垃圾收集高效，即其他工作线程停顿时间短（可控制在100ms内），只要垃圾收集发生的频率不高，完全可以接受。

### 1.3 使用的垃圾收集算法

复制 算法

### 1.4 应用场景

客户端模式下，虚拟机的 新生代区域

### 1.5 工作流程

![serial 垃圾收集器.png](https://upload-images.jianshu.io/upload_images/944365-7526204735d26a2b.png)

## 2. Serial Old收集器

### 2.1 定义

`Serial` 收集器 应用在老年代区域 的版本

### 2.2 优点

并发、单线程、效率高

> 同`Serial`收集器，此处不作过多描述

### 2.3 使用的垃圾收集算法

标记-整理 算法

### 2.4 应用场景

- 在客户端模式下，虚拟机的老年代区域
- 在服务器模式下：
  1. 与 `Parallel Scavenge` 收集器搭配使用
  2. 作为`CMS`收集器的后备预案，在并发收集发生`Concurrent Mode Failure`时使用

### 2.5 工作流程

![serial old 垃圾收集器.png](https://upload-images.jianshu.io/upload_images/944365-e19129b125d42b04.png)

## 3. ParNew 收集器

### 3.1 定义

`Serial` 收集器 的 多线程 版本。

### 3.2 优点

- 并发收集

在进行垃圾收集时，必须暂停其他所有工作线程（`Stop The World`），直到收集结束。

> 暂停工作线程 是在用户不可见的情况下进行

- 多线程收集

使用 **多条垃圾收集线程（`GC`线程）** 完成垃圾收集

> 由于存在线程交互的开销，所以在单`CPU`环境下，性能差于 `Serial` 收集器

- 与`CMS`收集器配合工作

目前，只有 `ParNew` 收集器能与 `CMS` 收集器 配合工作

> 1. 由于 `CMS` 收集器使用广泛，所以该特点非常重要。
2. 关于 `CMS` 收集器 下面会详细说明

### 3.3 使用的垃圾收集算法

复制算法

### 3.4 应用场景

服务器模式下，虚拟机的 新生代区域

> 多线程收集

### 3.5 工作流程

![parnew 垃圾收集器.png](https://upload-images.jianshu.io/upload_images/944365-32ee8abefdf522a5.png)

## 4. Parallel Scavenge收集器

### 4.1 定义

`ParNew` 收集器的升级版

### 4.2 特点

- 具备 ParNew` 收集器并发、多线程收集的特点
- 以达到 **可控制吞吐量** 为目标

其他收集器的目标是： **尽可能缩短 垃圾收集时间**，而`Parallel Scavenge`收集器的目标则是：**达到 可控制吞吐量**

> 1. 吞吐量：`CPU`用于运行用户代码的时间 与 `CPU`总消耗时间（运行用户代码时间+垃圾收集时间）的比值
> 2. 如：虚拟机总共运行100分钟，其中垃圾收集时间=1分钟、运行用户代码时间 = 99分钟，那吞吐量 = 99 / 100 = 99%

- 自适应

该垃圾收集器能根据当前系统运行情况，动态调整自身参数，从而达到最大吞吐量的目标。

> 1. 该特性称为：GC 自适应的调节策略
> 2. 这是`Parallel Scavenge`收集器与 `ParNew` 收集器 最大的区别

### 4.3 使用的垃圾收集算法

复制 算法

### 4.4 应用场景

服务器模式下，虚拟机的 新生代区域

### 4.5 工作流程

![parlleal 垃圾收集器.png](https://upload-images.jianshu.io/upload_images/944365-80a5f071ebd4261a.png)

## 5. Parallel Old收集器

### 5.1 定义

`Parallel Scavenge` 收集器 应用在老年代区域 的版本

### 5.2 特点

以达到 可控制吞吐量 为目标、自适应调节、多线程收集

> 同`Parallel Scavenge`收集器

### 5.3 使用的垃圾收集算法

标记-整理 算法

### 5.4 应用场景

服务器模式下，虚拟机的 老年代区域

### 5.5 工作流程

![parlleal old 垃圾收集器.png](https://upload-images.jianshu.io/upload_images/944365-4b955f124e559080.png)

## 6. CMS收集器

### 6.1 定义

即`Concurrent Mark Sweep`，基于 标记-清除算法的收集器

### 6.2 特点

#### 6.2.1 优点

- 并行

用户线程 & 垃圾收集线程同时进行。

> 即在进行垃圾收集时，用户还能工作。

- 单线程收集

只使用 一条线程 完成垃圾收集（GC线程）

- 垃圾收集停顿时间短

该收集器的目标是： **获取最短回收停顿时间** ，即希望 系统停顿的时间 最短，提高响应速度

#### 6.2.2 缺点

- 总吞吐量会降低

因为该收集器对`CPU`资源非常敏感，在并发阶段，虽不会导致用户线程停顿，但会因为占用部分线程（`CPU`资源）而导致应用程序变慢，总吞吐量会降低

- 无法处理浮动垃圾

由于 并发清理时 用户线程还在运行，所以会有新的垃圾不断产生（即浮动垃圾），只能等到留待下一次`GC`时再清理掉。

> 1. 因为这一部分垃圾出现在标记过程之后，所以CMS无法在当次GC中处理掉它们
> 2. 因此，CMS无法等到老年代被填满再进行Full GC，CMS需要预留一部分空间。即所谓的：可能出现Concurrent Mode Failure失败而导致另一次Full GC产生。

- 垃圾收集后会产生大量内存空间碎片

因为 CMS收集器是基于“标记-清除”算法的。

### 6.3 使用的垃圾收集算法

标记-清除 算法

### 6.4 应用场景

重视应用的响应速度、希望系统停顿时间最短的场景

> 如互联网移动端应用

### 6.5 工作流程

- `CMS` 收集器 是基于 标记-清除算法实现的收集器，工作流程较为复杂：（分为四个步骤）

1. 初始标记
2. 并发标记
3. 重新标记
4. 并发清除

![cms 工作流程.png](https://upload-images.jianshu.io/upload_images/944365-7b81e6a01d344764.png)

- 下面用一张图详细说明工作流程：

<img src="https://upload-images.jianshu.io/upload_images/944365-2e57ca743e39d59c.png" alt="cms 垃圾收集器.png" style="zoom:150%;" />

- 由于整个过程中，耗时最长的并发标记 和 并发清除过程都可与用户线程一起进行
- 所以，`CMS` **收集器的垃圾收集过程可看作是与用户线程 并发执行的**。

## 7. G1 收集器

### 7.1 定义

最新、技术最前沿的垃圾收集器

### 7.2 特点

- 并行

用户线程 & 垃圾收集线程同时进行。

> 即在进行垃圾收集时，用户还能工作

- 多线程

即使用 多条垃圾收集线程（`GC`线程） 进行垃圾收集

**并发 & 并行 充分利用多 `CPU` 、多核环境下的硬件优势 来缩短 垃圾收集的停顿时间**

- 垃圾回收效率高

G1 收集器是 针对性 对 Java堆内存区域进行垃圾收集，而非每次都对整个 Java 堆内存区域进行垃圾收集。

> 1. 即 G1收集器除了将 Java 堆内存区域分为新生代 & 老年代之外，还会细分为许多个大小相等的独立区域（ Region），然后G1收集器会跟踪每个 Region里的垃圾价值大小，并在后台维护一个列表；每次回收时，会根据允许的垃圾收集时间 优先回收价值最大的Region，从而避免了对整个Java堆内存区域进行垃圾收集，从而提高效率。
> 2. 因为上述机制，G1收集器还能建立可预测的停顿时间模型：即让 使用者 明确指定一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得从超出N毫秒。即具备实时性

- 分代收集

同时应用在 内存区域的新生代 & 老年代

- 不会产生内存空间碎片

  1. 从整体上看，G1 收集器是基于 标记-整理算法实现的收集器
  2. 从局部上看，是基于 复制算法 实现

上述两种算法意味着 G1 收集器不会产生内存空间碎片。

### 7.3 使用的垃圾收集算法

- 对于新生代：复制算法
- 对于老年代：标记 - 整理算法

### 7.4 应用场景

服务器端虚拟机的内存区域（包括 新生代 & 老年代）

### 7.5 工作流程

- G1 收集器的工作流程分为4个步骤：

1. 初始标记
2. 并发标记
3. 最终标记
4. 筛选回收

![g1 垃圾收集器 工作量流程.png](https://upload-images.jianshu.io/upload_images/944365-6d4bc33368471f19.png)

下面用一张图详细说明工作流程

<img src="https://upload-images.jianshu.io/upload_images/944365-a1b509c14799de40.png" alt="g1 垃圾收集器.png" />

## 8. 总结

<img src="https://upload-images.jianshu.io/upload_images/944365-a1b509c14799de40.png" alt="g1 垃圾收集器.png" style="zoom:150%;" />


### 参考

[https://www.jianshu.com/p/e5d2435a9122](https://www.jianshu.com/p/e5d2435a9122)