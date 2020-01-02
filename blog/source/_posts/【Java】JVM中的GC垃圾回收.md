---
title: 【Java】JVM中的GC垃圾回收
tags: [java]
date: 2020-1-02
---

# JVM中的GC垃圾回收
## JVM架构图

![](../img/jvm_conponent.png)

![](../img/jvm_memory3.png)

JVM被划分为三个主要的子系统
- 类装载子系统（Class Loader Subsystem）
- 运行时数据区（Runtime Data Area）
- 执行引擎（Execution Engine）

## STW - Stop The World
`Java`中`Stop-The-World`机制简称`STW`，是在执行垃圾收集算法时，`Java`应用程序的其他所有线程都被挂起（除了垃圾收集帮助器之外）。`Java`中一种全局暂停现象，全局停顿，所有`Java`代码停止，`native`代码可以执行，但不能与`JVM`交互。

除了`GC`，`JVM`下还会发生停顿现象:

`JVM`里有一条特殊的线程－－`VM Threads`，专门用来执行一些特殊的`VM Operation`，比如分派`GC`，`thread dump`等，这些任务，都需要整个`Heap`，以及所有线程的状态是静止的，一致的才能进行。所以`JVM`引入了安全点(`Safe Point`)的概念，想办法在需要进行`VM Operation`时，通知所有的线程进入一个静止的安全点。

除了`GC`，其他触发安全点的`VM Operation`包括：
- 1. JIT相关，比如`Code deoptimization, Flushing code cache`；
- 2. `Class redefinition (e.g. javaagent，AOP代码植入的产生的instrumentation)`；
- 3. `Biased lock revocation` 取消偏向锁；
- 4. `Various debug operation (e.g. thread dump or deadlock check)`；

### 参考

[现代JVM中的Safe Region和Safe Point到底是如何定义和划分的?](https://www.zhihu.com/question/29268019/answer/43762165)

## JVM Garbage Collectors


![](../img/gc_collection.jpg)

|  收集器   | 收集范围  | 算法  | 执行类型  |备注  |
|  ----  | ----  |----  |----  |----  |
| `Serial(串行GC)`  | 新生代 |复制 |单线程 |Jvm client模式下默认的新生代收集器|
| `ParNew(并行GC)`  | 新生代 |复制 |多线程并行 |serial收集器的多线程版本|
| `Parallel Scavenge(并行回收GC)`  | 新生代 |复制 |多线程并行 |目标是达到一个可控制的吞吐量<br>吞吐量= 程序运行时间/(程序运行时间 + 垃圾收集时间)<br>虚拟机总共运行了100分钟。其中垃圾收集花掉1分钟，那吞吐量就是99%。|
| `Serial Old(串行GC)`  | 老年代 |标记整理 |单线程 |Serial收集器的老年代版本<br>主要使用在Client模式下的虚拟机|
| `Parallel Old(并行GC)` | 老年代 |标记整理 |多线程并行 |Parallel Scavenge收集器的老年代版本|
| `CMS(并发GC)` | 老年代 |标记清除 |多线程并发 |以获取最短回收停顿时间为目标的收集器|
| `G1` | 全部 |复制算法,标记整理 |多线程 |以获取最短回收停顿时间为目标的收集器|

> `并行(Parallel)`：多条垃圾收集线程并行工作，而用户线程仍处于等待状态  
> `并发(Concurrent)`：垃圾收集线程与用户线程一段时间内同时工作(交替执行)

### 参考
[JVM GC参数以及GC算法的应用](https://my.oschina.net/hosee/blog/644618)

[JVM GC算法](https://cloud.tencent.com/developer/article/1102243)

[java虚拟机之HotSpot垃圾收集器](https://www.toutiao.com/i6490796229067276814/)

[Java虚拟机学习 - 垃圾收集器](https://blog.csdn.net/java2000_wl/article/details/8030172)

