---
title: "Garbage-First Garbage Collection"
date: 2026-03-16
draft: false
description: "G1GC是sun公司创新的一种用于java语言的GC算法"
tags: ["example", "tag"]
---
{{< katex >}}

# Garbage-First Garbage Collection

## 论文汇报 | 2026年3月

- 服务器风格GC
- 针对多核大内存机器，
- 满足软即时性和高吞吐目标。
- 利用并发避免过长停顿

---

# 目录

- 问题背景与概述
- 数据结构
- 核心机制1：驱逐暂停
- 核心机制2：并发标记
- 启发式方法
- 部分benchmark
- 总结

---

# 问题背景与概述

## 背景

Java被广泛用来做大型服务器应用。
这些应用有着以下特点：
大量存活堆数据
相当量线程级并发
在高端多处理器机器上运行

对这样的软件吞吐量很重要，同时他们还有严苛的（尽管是弹性的）即时性约束。

$$吞吐量=用户代码执行的时间/(用户代码执行的时间+GC时间)$$

## 问题

Java语言通过GC来收割垃圾内存。
传统stop-world GC实现会影响应用反应速度，
而并发式或增量式GC中，更短的停顿时间又通常伴随着吞吐量的代价

## 问题

G1GC允许用户指定一个软即时目标，期望GC在任意y ms的时间片中消耗不超过x ms的时间。
通过显式声明目标，GC可以尽力保证GC停顿时间短和频率小来满足应用的要求，同时又不导致降低吞吐或增加不必要的负担。

G1GC尝试在大堆，高分配率，运行在多核机器的应用上满足这样的一个软即时目标，同时维持高吞吐。

## G1GC的数据结构

G1将堆划分成一组等大的heap regions，并会记录下region间双向的引用关系，这使得G1GC可以选择任何一个区域进行收割

记录引用关系的方式是对于任一个region，它维护一个它的remembered set，RS会记录来自其它所有region（分代法中的young regions除外）指向改region的指针（的大约位置）。

变异器线程会创建log records，一个并发线程处理这些log records来更新RS。

## G1GC的并发标记

G1GC的并发标记算法使用SATB策略，提供可靠的对象存活性数据和每个区域的存活数据量分析。这为选择收割有更少的存活数据和更少的收割成本的region提供信息。

## G1GC的即时性

G1GC采取粒度为region的可打断性，不保证硬即时，但比粒度为拷贝一个对象的硬性即使GC有更好的吞吐和空间利用，这对于大部分软件来说是好的tradeoff

---

# 数据结构

## 堆

heap region：G1堆被分成等大的regions，每一个region都是连续的虚拟内存空间。

current allocation region：正在分配空间的region。

TLAB(thread-local allocation buffers)：变异器线程在堆区域上通过CAS预先分配的一块私有空间，后面变异器线程在自己的TLAB上分配对象，避免了竞争。

空region被组织进链表，当前region满了以后可以在常数时间获得下一个region。

超过region大小3/4的对象，会被放进专门的区域

## Remembered Set

remembered set(RS)：每个region有一个RS,指示所有region外可能有指针指向region内活对象的位置。

card table: 堆里的每512字节空间叫一个卡片。每个卡片被映射到卡表里1字节的卡项。

remembered set log（RS log）：每个线程局部的脏卡片缓冲区

filled RS buffers：全局脏卡片的集合。

RS本身是卡片的哈希表，由于并发，每个region有一组RS，每个线程一个哈希表，来允许各线程不互相干涉的更新RS。

RS的逻辑内容是每个region的一组RS的并。

```js
1| rTmp := rX XOR rY
2| rTmp := rTmp >> LogOfHeapRegionSize
3| // Below is a conditional move instr
4| rTmp := (rY == NULL) then 0 else rTmp
5| if (rTmp == 0) goto filtered
6| call rs_enqueue(rX)
7| filtered:
```

在指针写后会触发**RS写屏障**

RS写屏障的目的是对x.f = y，若x,y不属于同一region(1，2行)且y != null(4，5行)，并发GC线程就把x相应的卡片dirty掉（如果未dirty），然后指向这张卡片的指针放入该线程RS log中。

当线程的RS log填满，整个RS log buffer会被加入到全局的filled RS buffer，然后线程创建一个新的RS log buffer。

并发RS线程发现filled RS buffer大小超过初始阈值，就开始处理filled buffer，直到filled buffer大小小于初始阈值的1/4。处理的过程是，对每一个log buffer，一张一张处理里面的卡表项。而对每一张卡表项，先将其设为clean，以便在处理期间它被re-dirty，然后检查卡片内所有对象的指针域，如果发现其中有指向其他region的指针，就把卡片插入到被指向region的RS中。

热卡片处理：一些卡片范围内的指针域频繁写入其他区域对象地址，把他们称作热卡片。我们希望识别热卡片并避免反复处理他们。我们使用另一张卡片哈希表记录卡片自上次驱逐暂停后的dirty次数，当次数超过阈值，就把它加入热队列，其中的卡片不会被并发期间处理，而是延迟到下次驱逐暂停的开始被统一处理。如果热队列满了，就选一个最不热门的卡片evict出去。

当RS buffer增长过快，并发线程跟不上，超过一定大小时，尝试继续添加buffer的变异器线程需要自己处理RS。

---

# 核心机制1：驱逐暂停

collection set：我们选择要在驱逐暂停阶段清理的区域。

简述：
在一些时机（见启发式部分）进行驱逐暂停。
这一阶段，驱逐collection set包含的区域，将这些区域的活对象驱逐到其他区域，然后释放这些区域。
我们选择尽可能并发处理这一阶段。
第一步是串行地选择collection set（见启发式部分）

然后，多个GC线程并发地做三件事：
扫描待处理的RS buffer来更新RS，
扫描RS和其它root groups中的活对象，
驱逐活对象。

为了达到更快的并行分配，使用GCLAB，线程分配一个需要驱逐的对象的拷贝到它的GCLAB中，然后在老对象位置安装一个转发指针。成功安装转发指针的线程负责拷贝和扫描内容并处理。

使用基于工作窃取的技术提供负载均衡。

## 分代GC

当变异器线程分配region时，启发式地分配young regions。使得young regions一定被包含进下一次驱逐暂停的collection set。

这样我们就不必处理young region指向其他region的RS log，而在驱逐暂停时直接处理其中的引用关系。

除此之外young region和non-young region没有其他区别

G1提供了分代模式和纯G1模式
分代模式有fully young和partially young两个模式，
fully young只处理所有young regions，
partially young同时处理所有young regions和一些专门选取的non-young regions。

---

# 核心机制2：并发标记

并发标记提供了对象的存活性信息和region的存活数据量等统计信息。

SATB: snapshot-at-the-beginning。
标记算法保证提供的信息是基于标记工作开始时的对象图情况。
标记期间新分配的对象被认为是存活的，不构成对象图的一部分。

对于bitmap和TAMS，都维护previous和next两组信息。
previous是上次并发标记完成的结果信息，驱逐暂停等过程可以直接利用这些信息。
next是并发标记正在构建的信息。
他们会在标记结束时交换角色

逻辑上
previous next
  丢弃    previous next

## 数据结构

使用bitmap来指示对象存活性信息，1bitmap比特可以映射64堆比特。
使用mark stack来放置灰对象（标记了但没有扫描完）。

每个堆维护TAMS(top at mark start)，指示当前标记过程的范围，
TAMS以上的对象是标记期间分配的对象，认为它们被隐式标记了。

## 初始标记暂停

先并发地清空next bitmap。
停止所有变异器线程
初始标记暂停开始：
标记所有从根直达的对象。
遍历所有堆区域，令每个区域 $nextTAMS=区域top$。

变异器线程重新启动

并发标记开始：

堆遍历的优化：Finger遍历bitmap里被标记的bit，高于finger的对象是灰色，低于finger的会对象被放进标记栈

## 并发标记写屏障

变异器有可能在正在并发标记时更改指针图，破坏SATB保证。因此在指针写之前添加并发标记写屏障

```js
1| rTmp := load(rThread + MarkingInProgressOffset)
2| if (!rTmp) goto filtered
3| rTmp := load(rX + FieldOffset)
4| if (rTmp == null) goto filtered
5| call satb_enqueue(rTmp)
6| filtered:
```

当进行
\[rX,FieldOffset\]=rY
如果在并发标记阶段，且rX+FieldOffset位置不为null，就把rX+FieldOffset的值加入到当前线程的标记缓冲区，当标记缓冲区满，就被加入到全局标记缓冲区，然后并发标记线程定期检查集合大小，打断堆遍历过程来处理填满的缓冲区。

## 最终标记暂停

停止所有变异器线程，多线程地处理所有的未被处理的全局标记缓冲区里的log和线程局部缓冲区里的log。

## 存活数据计数和清理

当最终标记暂停完成时，存活数据计数会并发地进行，GC线程重新检查每个region，通过bitmap计数区域内的被标记数据的量。

并发标记期间发生的驱逐暂停会增加next TAMS的值，cleanup暂停可靠的终止计数过程。
这一阶段next和previous bitmap和TAMS交换身份

最后cleanup阶段按照GC效益给各个regions排序

不包含存活数据的区域会在这个阶段立刻被清理。

## 驱逐暂停和标记的交互

1. 驱逐暂停不会回收标记阶段标记的死对象

2. 驱逐一个对象时，必须保证标记正确，这个对象在previous marking中是活的，且必须让next marking知道其新的位置变更。

3. 必须允许在并发标记进行时发生驱逐暂停，为了做到这一点，把标记栈作为根的来源之一。

## 热门对象处理

热门对象：被很多其他位置引用的对象

在堆区域中预留一个前缀区专门放热门对象，尝试快速地识别热门对象，然后把它们放在前缀区。而前缀区的这些regions不会被回收。

RS大小达到给定阈值的regions触发热门暂停，热门暂停在region的每个对象上安放大约的引用计数，引用计数快速达到阈值的对象会被单独驱逐到前缀区，非流行对象驱逐到堆的普通部分。如果没有发现热门对象，就没有驱逐发生，但区域的阈值翻倍。

修改RS写屏障

```js
if (rY < PopObjBoundary) goto filtered
```

这样，若rY为null或rY为热门区对象，指向它的卡片将不被记录进log

---
{{< katex >}}

# 启发式方法

用户输入：
1. 内存空间使用的上界
2. 软即时目标：在每个x ms时间片中，GC只能消耗不超过y ms的GC时间
3. 是否使用分代模式

## 实现软即时目标

1. 保证单次暂停不超过暂停时间的上界
2. 调度暂停防止短时间内过多次暂停

### 预测暂停时间

对fully-young模式，我们分析一次驱逐暂停多少young regions能产生想要的时间间隔。两次驱逐暂停期间能分配多少young regions。
而partially-young模式，如果暂停时间允许的话，可能添加更多non-young区域加入收割，在这个模式下和纯G1模式下，
如果即使选择最好的区域也会超过暂停时间界限，我们就停止选择区域。

对于后者
$$V(cs)=V_{fixed} + U * d + \Sigma_{r\sub cs} (S · rsSize(r) + C · liveBytes(r))$$

\( V(cs)\) is the cost of collecting collection set cs;
\(V_{fixed}\) represents fixed costs, common to all pauses;
\(U\) is the average cost of scanning a card, and \(d\) is the
number of dirty cards that must be scanned to bring
remembered sets up-to-date;
\(S\) is the of scanning a card from a remembered set for
pointers into the collection set, and \(rsSize(r)\) is the
number of card entries in r's remembered set; and
\(C\) is the cost per byte of evacuating (and scanning)
a live object, and \(liveBytes(r)\) is an estimate of the
number of live bytes in region r.

\(V_{fixed}\),\(U\) ,\(S\),\(C\)，是比较恒定的，我们一开始采用一个初始估计，然后在实际运行时动态修正。

引入confidence置信值，以给定标准差的倍数调整参数。

剩下的几个变量是在驱逐暂停开始时计算出来的，
对于liveBytes来说如果一个区域包含的对象上次在并发标记开始时就分配好了。上次并发标记就提供了一个存活数据的上界。
而对于上次标记之后新分配的区域，可以动态估计存活率，然后用存活率计算预期的存活数据量。还可以跟踪存活率方差，根据置信值调整。

### 调度暂停

只要有充足的内存空间，我们就可以延迟任何标记暂停和驱逐暂停。

当年轻代的驱逐暂停不得不被推迟时，我们可以让变异器线程把对象分配到老年代区域。

使用一个start/stop时间对队列，处理最近发生的暂停时间片，从一段插入暂停，从另一端丢弃已经离开时间片的无关时间对。
我们可以知道现在最多能暂停多久，以及多久之后我们才能暂停一个已知时间长度的暂停。

## 选择collection set

$$预期GC效率 = 估计的垃圾量/估计的回收成本$$

在标记结束时我们获得了初步的效益估计以及排序，但是这会随时间改变，所以在驱逐暂停开始时，我们选取初步排序中固定大小的前缀依据当前的效益估计进行重新排序。然后选择区域，当时间超过或空间不够时停止选择。

## 何时发起驱逐暂停

选择一个硬边际变量h，假设堆大小为M，
有硬极限$H = (1-h)*M$，硬极限就是我们至少要保证堆中有H的空间，否则进行驱逐时会发生没有空间驱逐的问题。当分配的空间到达了应边界，必须发起驱逐暂停。

在fully-young模式下，维护一个关于有多少个年轻代region时进行驱逐暂停能满足暂停时间的动态估计，当年轻代区域数量达到估计时发起驱逐暂停。
在partially-young模式下，只要软即时目标允许，我们就发起驱逐暂停。

分代模式一开始是fully-young，一个并发标记之后转向partially-young模式，然后当partially-young模式效益降低到fully-young一致时转回fully-young模式。但是当堆快满时，即使效益降低，也必须进行partially-young来降低堆占用

## 何时发起并发标记

维护一个软边界u
定义软极限$ H - uM = M - hM - uM $
在驱逐暂停前，当堆占用达到软极限，只要软即时允许，标记就会被发起。

---

# 总结与未来工作

## G1 的三大核心贡献

- **避免碎片化**
  - 首个高性能服务器收集器，通过充分压缩避免细粒度空闲列表
  - 简化内存管理，基本消除碎片问题
- **智能选区策略**
  - 基于全局标记信息和 GC 效率公式（垃圾量/成本）优先选择区域
  - 而非仅依据存活数据量，实现暂停时间目标
- **软实时调度**
  - 使用滑动窗口队列跟踪历史暂停，调度未来 GC
  - 比硬实时更灵活，允许时间片内多次不同长度的暂停

## 未来优化方向

- **写屏障优化**：改进 remembered set 表示，提高处理效率
- **静态分析**：探索在编译期移除部分写屏障的可能性
- **逃逸分析**：结合编译时逃逸分析，进一步优化对象分配
