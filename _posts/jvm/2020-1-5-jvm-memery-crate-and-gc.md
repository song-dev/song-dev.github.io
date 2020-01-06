---
layout: post
title: "内存分配与回收策略"
subtitle: ' JVM 相关知识 '
author: "Song"
header-style: text
tags:
  - java
  - JVM
  - gc
---

## 背景

- 对象分配大部分就是在堆上分配
- 对象主要分配在新生代的 Eden 区上，少数大对象可能直接分配在老年代
- 对象分配方案并不是固定的，其取决于哪一种垃圾收集器组合，还有虚拟机参数配置

## 1. 对象优先在 Eden 分配

- 新生代内存区域分为 Eden、From Survivor、To Survivor，其大小比例为 8:1:1，直接可用的为 Eden 和 From Survivor 区域
- 大多数情况下，对象在新生代 Eden 区中分配，当 Eden 区没有足够空间时，虚拟机将发起一次 Minor GC

> - Minor GC(新生代 GC)：指发生在新生代的垃圾收集动作，因为 java 对象大多都具备朝生夕灭特性，所以 Minir GC 非常频繁，一般回收速度很快
> - Full GC/Major GC(老年代 GC)：发生在老年代的 GC，出现了Major GC，经常回伴随一次 Minor GC。Minor GC的速度一般比 Major GC 快 10 倍以上

- 代码示例

```java
private final static int _1MB = 1024 * 1024;

/**
 * -verbose:gc -Xms20M -Xmx20M -Xmn10M XX:+PrintGCDetails -XX:+SurvivorRation=8
 * 测试新生代 GC
 */
@Test
public void test_minor_gc() {
    byte[] allocation1, allocation2, allocation3, allocation4;
    allocation1 = new byte[2 * _1MB];
    allocation2 = new byte[2 * _1MB];
    allocation3 = new byte[2 * _1MB];
    // 出现一次 Minor gc
    allocation4 = new byte[4 * _1MB];
    // 主动 GC
    System.gc();
}
```

- gc 日志

```
[GC (Allocation Failure) [PSYoungGen: 8192K->1008K(9216K)] 8192K->1160K(19456K), 0.0033070 secs] [Times: user=0.01 sys=0.01, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 9200K->992K(9216K)] 9352K->1384K(19456K), 0.0021295 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] [GC (Allocation Failure) [PSYoungGen: 7659K->992K(9216K)] 8050K->3583K(19456K), 0.0049381 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 
[GC (System.gc()) --[PSYoungGen: 5284K->5284K(9216K)] 11971K->14027K(19456K), 0.0024733 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 
[Full GC (System.gc()) [PSYoungGen: 5284K->2998K(9216K)] [ParOldGen: 8743K->8227K(10240K)] 14027K->11225K(19456K), [Metaspace: 5047K->5047K(1056768K)], 0.0085891 secs] [Times: user=0.02 sys=0.00, real=0.01 secs] 
Heap
 PSYoungGen      total 9216K, used 3322K [0x00000007bf600000, 0x00000007c0000000, 0x00000007c0000000)
  eden space 8192K, 40% used [0x00000007bf600000,0x00000007bf93e8e8,0x00000007bfe00000)
  from space 1024K, 0% used [0x00000007bfe00000,0x00000007bfe00000,0x00000007bff00000)
  to   space 1024K, 0% used [0x00000007bff00000,0x00000007bff00000,0x00000007c0000000)
 ParOldGen       total 10240K, used 8227K [0x00000007bec00000, 0x00000007bf600000, 0x00000007bf600000)
  object space 10240K, 80% used [0x00000007bec00000,0x00000007bf408dc8,0x00000007bf600000)
 Metaspace       used 5057K, capacity 5264K, committed 5504K, reserved 1056768K
  class space    used 574K, capacity 627K, committed 640K, reserved 1048576K

Process finished with exit code 0
```

- 6M 被分配在新生代，当要分配额外的 4M 时候新生代内存空间不够，且 To Survivor 空间只有 1M，故分配担保失败，6M 对象被迁移到 老年区，4M 对象被分配在 Eden 区域

## 2. 大对象直接进入老年代

- 大对象指的是大量连续内存空间的 java 对象，如数组、长字符串

> 应该极力避免出现朝生夕灭的大对象，容易导致内存有不少空间就提前触发 GC，而获得连续空间分配给另外的大对象

- 对于 Serial 、ParNew 收集器 可设置 -XX:PretenureSizeThreshold 参数，令比设置的对象大的内存直接在老年代分配

> 直接在老年代分配可以避免在 Eden区以及两个 Survivor 区之间发生大量的内存复制

## 3. 长期存活的对象直接进入老年代

### 3.1 对象年龄定义

虚拟机给每个对象定义了一个年龄计数器，如果对象在 Eden 区出生且经过第一次 Minor GC 后仍然存活，且能被 Survivor 容纳，将被移动到 Survivor 空间中，且对象年龄设置为1。

### 3.2 条件

- 对象在 Survivor 区中每经过一次 Minor GC ，年龄就增加一岁
- 当对象的年龄增加到15岁(默认)，对象就会被晋升到老年代中

> 可以通过 -XX:MaxTenuringThreshold=15 设置对象晋升年龄阈值

## 4. 动态对象年龄判断

- 虚拟机并不是永远要求对象的年龄必须达到阈值才能晋升到老年代

### 4.1 条件

- 如果在 Survivor 空间中，相同的年龄所有对象大小的综合大于 Survivor 空间的一半，年龄大于或者等于该年龄的对象可直接进入老年代，无需等到达到阈值年龄

## 5. 空间分配担保

### 5.1 背景

- **在发生 Minor GC 之前**，虚拟机会先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果这个条件成立，则当前 Minor GC 是安全的
- 如果不成立，这虚拟机会查看 HandlePromotionFailure 设置值是否允许担保失败(默认允许)
- 如果允许，那么会继续检查老年代最大可用连续空间是否大于历次晋升到老年代对象平均大小
- 如果大于则尝试进行一次 Minor GC(可能有风险)
- 如果小于，或者 HandlePromotionFailure 设置为不允许担保失败，那么此时改为一次 Full GC

### 5.2 担保失败

- 当 Minor GC 之后，假如新生代大量对象存活，则需要老年代分配担保，防止 Survivor 区无法容纳对象直接进入老年代，老年代必须要容纳这些对象剩余空间
- 需要将之前进入老年代对象平均值大小，与老年代剩余空间比较，决定是否 Full GC 让老年代腾出更多空间
- 取平均值比较，仍然可能担保失败，若担保失败，则在失败后重新发起一次 Full GC
- jdk 1.6 之后 HandlePromotionFailure 参数已经未使用，且规则更新为 只要老年代的连续空间大于新生代对象总大小或者历次晋升的平均值就会 Minor GC，否则 Full GC

## 6. 总结

- 堆 GC 流程图

<img src="https://song-dev.github.io/img/in-post/post-jvm/jvm-gc流程图.jpg" alt="jvm-gc流程图" style="zoom:70%;" />

> 以 jdk 1.6 update 24 以上为准，空间分配担保失败开关永远为开

- Eden 或者 From Survivor 区对象经过一次 Minor GC ，若还存活，则被赋值到 To Survivor 区域，且清除 Eden 和 From Survivor 区域
- To Survivor 区对象，达到一定年龄进入老年代
- 新生代区域对象总共大小若小于老年代最大连续空间，则直接安全 Minor GC
- 新生代区域晋升老年代平均大小，若小于老年代最大连续空间，则直接 Minor GC，这时担保失败则 Full GC。防止 To Survivor 区域接收不了 Eden 区域对象，老年代担保失败，Full GC 增加空闲空间
- 新生代区域晋升老年代平均大小，若大于老年代最大连续空间，则直接 Full GC，增加空闲区域为新生代担保
- Full GC 可能存在 STW 问题，应该极力避免

