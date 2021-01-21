---
layout: post
title: "HashMap 为什么线程不安全？"
subtitle: '从散列函数、哈希冲突、扩容方案、线程安全四部分分析'
author: "Song"
header-style: text
tags:
  - java
  - collection
  - HashMap
  - security
---

- 1.jdk1.7中的HashMap

- - 1.1 扩容造成死循环分析过程
  - 1.2 扩容造成数据丢失分析过程

- 2.jdk1.8中HashMap

- 总结

------

前言：我们都知道HashMap是线程不安全的，在多线程环境中不建议使用，但是其线程不安全主要体现在什么地方呢，本文将对该问题进行解密。

# 1.jdk1.7中的HashMap

在jdk1.8中对HashMap做了很多优化，这里先分析在jdk1.7中的问题，相信大家都知道在jdk1.7多线程环境下HashMap容易出现死循环，这里我们先用代码来模拟出现死循环的情况：

```
1publicclass HashMapTest {
 2
 3public static void main(String[] args) {
 4         HashMapThread thread0 = new HashMapThread();
 5         HashMapThread thread1 = new HashMapThread();
 6         HashMapThread thread2 = new HashMapThread();
 7         HashMapThread thread3 = new HashMapThread();
 8         HashMapThread thread4 = new HashMapThread();
 9         thread0.start();
10         thread1.start();
11         thread2.start();
12         thread3.start();
13         thread4.start();
14     }
15 }
16
17class HashMapThread extends Thread {
18privatestatic AtomicInteger ai = new AtomicInteger();
19privatestatic Map<Integer, Integer> map = new HashMap<>();
20
21@Override
22public void run() {
23while (ai.get() < 1000000) {
24             map.put(ai.get(), ai.get());
25             ai.incrementAndGet();
26         }
27     }
28 }
```

上述代码比较简单，就是开多个线程不断进行put操作，并且HashMap与AtomicInteger都是全局共享的。在多运行几次该代码后，出现如下死循环情形：

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfecWQvk4vhdQnUBzictjvKTVxfrLh47NsI3vlnlEm5SZoFntDGHyOGiagBTkicHEoXicMCichfzmpQd77A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

其中有几次还会出现数组越界的情况：

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfecWQvk4vhdQnUBzictjvKTVkkdL1xS8Zuflqialg1tHicODicKedMueS4ia329NNOWT2DSXEYwib1udiciaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这里我们着重分析为什么会出现死循环的情况，通过jps和jstack命名查看死循环情况，结果如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfecWQvk4vhdQnUBzictjvKTVYfNR0Cib1kM0z01oBTt06pG9YPmIT7CGibF8g11paEQqPicf7Z3WSZQfA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

从堆栈信息中可以看到出现死循环的位置，通过该信息可明确知道死循环发生在HashMap的扩容函数中，根源在**transfer函数**中，jdk1.7中HashMap的transfer函数如下：

```
1void transfer(Entry[] newTable, boolean rehash) {
 2int newCapacity = newTable.length;
 3for (Entry<K,V> e : table) {
 4while(null != e) {
 5                 Entry<K,V> next = e.next;
 6if (rehash) {
 7                     e.hash = null == e.key ? 0 : hash(e.key);
 8                 }
 9int i = indexFor(e.hash, newCapacity);
10                 e.next = newTable[i];
11                 newTable[i] = e;
12                 e = next;
13             }
14         }
15     }
```

总结下该函数的主要作用：

在对table进行扩容到newTable后，需要将原来数据转移到newTable中，注意10-12行代码，这里可以看出在转移元素的过程中，使用的是头插法，也就是链表的顺序会翻转，这里也是形成死循环的关键点。下面进行详细分析。

## 1.1 扩容造成死循环分析过程

前提条件：

这里假设

\#1.hash算法为简单的用key mod链表的大小。

\#2.最开始hash表size=2，key=3,7,5，则都在table[1]中。

\#3.然后进行resize，使size变成4。

未resize前的数据结构如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfecWQvk4vhdQnUBzictjvKTV8KojY8VrMVmMVK0mT4Ric2icyc3icUzVuQCuLGpzOgZxOeHd8MgfgNuXQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如果在单线程环境下，最后的结果如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfecWQvk4vhdQnUBzictjvKTVCB9USBX1WYQPWzj7iciaTjlyHZpq2MlWRasFGPLvXEM1zf0tiaictibYSsg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这里的转移过程，不再进行详述，只要理解transfer函数在做什么，其转移过程以及如何对链表进行反转应该不难。

然后在多线程环境下，假设有两个线程A和B都在进行put操作。线程A在执行到transfer函数中第11行代码处挂起，因为该函数在这里分析的地位非常重要，因此再次贴出来。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfecWQvk4vhdQnUBzictjvKTVRtdI81vOOgiayBNwiaru5saibodJiaUAxBsic9Clib8qu2SZWQd6MlokUF9g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

此时线程A中运行结果如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfecWQvk4vhdQnUBzictjvKTVCrq6iaOLPVE0kE1zgCQYmWKTUPBGAm1icoUUR8ADvhUwTLoQ0jZzBr1g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

线程A挂起后，此时线程B正常执行，并完成resize操作，结果如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfecWQvk4vhdQnUBzictjvKTVGWwiboVcEoovrdg7qUibUcwrLTuww3rbptiaFoZI11NaTj0tDAxybhwag/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**这里需要特别注意的点：由于线程B已经执行完毕，根据Java内存模型，现在newTable和table中的Entry都是主存中最新值：7.next=3，3.next=null。**

此时切换到线程A上，在线程A挂起时内存中值如下：e=3，next=7，newTable[3]=null，代码执行过程如下：

```
newTable[3]=e ----> newTable[3]=3
e=next ----> e=7
```

此时结果如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfecWQvk4vhdQnUBzictjvKTVSPJ6RwE1NHxILdUiaIFib3NHncYPI6hfE0NvKWKu0UANfsFsWPicWKOvA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

继续循环：

```
e=7
next=e.next ----> next=3【从主存中取值】
e.next=newTable[3] ----> e.next=3【从主存中取值】
newTable[3]=e ----> newTable[3]=7
e=next ----> e=3
```

结果如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfecWQvk4vhdQnUBzictjvKTVlVDoSg5Kvo0maS6h9RNZVRd7sMsZtu5homM7KI9ibEk47WtJXibgXfcg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

再次进行循环：

```
e=3
next=e.next ----> next=null
e.next=newTable[3] ----> e.next=7 即：3.next=7
newTable[3]=e ----> newTable[3]=3
e=next ----> e=null
```

注意此次循环：e.next=7，而在上次循环中7.next=3，出现环形链表，并且此时e=null循环结束。

结果如下：

**![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfecWQvk4vhdQnUBzictjvKTVU7APPu5l0lKpFRwBS0KFISCE1h9iarAQnv7zpXz2k9DnZPIr2AWqhiaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)**

在后续操作中只要涉及轮询hashmap的数据结构，就会在这里发生死循环，造成悲剧。

## 1.2 扩容造成数据丢失分析过程

遵照上述分析过程，初始时：

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfecWQvk4vhdQnUBzictjvKTV9qbl5IdZHTh075WLV8eZpGiaHBY2u0baTcwTg0Y27N3icKIYQofQRcrA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

线程A和线程B进行put操作，同样线程A挂起：

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfecWQvk4vhdQnUBzictjvKTVRtdI81vOOgiayBNwiaru5saibodJiaUAxBsic9Clib8qu2SZWQd6MlokUF9g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

此时线程A的运行结果如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfecWQvk4vhdQnUBzictjvKTV0mE04nJPQHmSiaJpjVgV961cZOHvNVImchDLqzlC63WCAIiap5HWKZRQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

此时线程B已获得CPU时间片，并完成resize操作：

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfecWQvk4vhdQnUBzictjvKTV8icvQkHlaA3y2IK4flVgmprvIjuXEcIZaqiawTfpPZAw9R1xwz8ICoEg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

同样注意由于线程B执行完成，newTable和table都为最新值：**5.next=null**。

此时切换到线程A，在线程A挂起时：**e=7，next=5，newTable[3]=null。**

执行newtable[i]=e，就将**7放在了table[3]**的位置，此时next=5。接着进行下一次循环：

```
e=5
next=e.next ----> next=null，从主存中取值
e.next=newTable[1] ----> e.next=5，从主存中取值
newTable[1]=e ----> newTable[1]=5
e=next ----> e=null
```

将5放置在table[1]位置，此时e=null循环结束，**3元素丢失**，并形成**环形链表**。并在后续操作hashmap时造成死循环。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JdLkEI9sZfecWQvk4vhdQnUBzictjvKTVFF64CbJ5GTu4NxhPBpheia6wmdTncUGeQ4YGdNuG4BknRiaCHZw6Hxsg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 2.jdk1.8中HashMap

在jdk1.8中对HashMap进行了优化，在发生hash碰撞，不再采用头插法方式，而是直接插入链表尾部，因此不会出现环形链表的情况，但是在多线程的情况下仍然不安全，这里我们看jdk1.8中HashMap的put操作源码：

```
1final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
 2                    boolean evict) {
 3         Node<K,V>[] tab; Node<K,V> p; int n, i;
 4if ((tab = table) == null || (n = tab.length) == 0)
 5             n = (tab = resize()).length;
 6if ((p = tab[i = (n - 1) & hash]) == null) // 如果没有hash碰撞则直接插入元素
 7             tab[i] = newNode(hash, key, value, null);
 8else {
 9             Node<K,V> e; K k;
10if (p.hash == hash &&
11                 ((k = p.key) == key || (key != null && key.equals(k))))
12                 e = p;
13elseif (p instanceof TreeNode)
14                 e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
15else {
16for (int binCount = 0; ; ++binCount) {
17if ((e = p.next) == null) {
18                         p.next = newNode(hash, key, value, null);
19if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
20                             treeifyBin(tab, hash);
21break;
22                     }
23if (e.hash == hash &&
24                         ((k = e.key) == key || (key != null && key.equals(k))))
25break;
26                     p = e;
27                 }
28             }
29if (e != null) { // existing mapping for key
30                 V oldValue = e.value;
31if (!onlyIfAbsent || oldValue == null)
32                     e.value = value;
33                 afterNodeAccess(e);
34return oldValue;
35             }
36         }
37         ++modCount;
38if (++size > threshold)
39             resize();
40         afterNodeInsertion(evict);
41returnnull;
42     }
```

这是jdk1.8中HashMap中put操作的主函数， 注意第6行代码，如果没有hash碰撞则会直接插入元素。如果线程A和线程B同时进行put操作，刚好这两条不同的数据hash值一样，并且该位置数据为null，所以这线程A、B都会进入第6行代码中。假设一种情况，线程A进入后还未进行数据插入时挂起，而线程B正常执行，从而正常插入数据，然后线程A获取CPU时间片，此时线程A不用再进行hash判断了，问题出现：线程A会把线程B插入的数据给**覆盖**，发生线程不安全。

这里只是简要分析下jdk1.8中HashMap出现的线程不安全问题的体现，后续将会对java的集合框架进行总结，到时再进行具体分析。

# 总结

首先HashMap是**线程不安全**的，其主要体现：

\#1.在jdk1.7中，在多线程环境下，扩容时会造成环形链或数据丢失。

\#2.在jdk1.8中，在多线程环境下，会发生数据覆盖的情况。

> 转载 https://mp.weixin.qq.com/s?__biz=MzAwNDA2OTM1Ng==&mid=2453141934&idx=1&sn=49c4f2cef4d93f45c82886b254517bfb&scene=21#wechat_redirect