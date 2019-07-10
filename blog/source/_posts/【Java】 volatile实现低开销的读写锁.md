---
title: 【Java】 volatile实现低开销的读写锁
tags: [java]
date: 2019-7-10
---

# volatile实现低开销的读写锁






如果读操作远远超过写操作，您可以结合使用内部锁和 volatile 变量来减少公共代码路径的开销。

如下显示的线程安全的计数器，使用 `synchronized `确保增量操作是原子的，并使用 `volatile` 保证当前结果的可见性。

如果更新不频繁的话，该方法可实现更好的性能，因为读路径的开销仅仅涉及` volatile `读操作，这通常要优于一个无竞争的锁获取的开销。


**CODE:**

```java
public class Counter {

    /**
     * 低开销的读写锁技巧
     * 所有写操作必须在保持this锁的情况下完成
     * <p>
     * Employs the cheap read-write lock trick
     * All mutative operations MUST be done with the 'this' lock held
     */
    private volatile int value;

    /**
     * 读操作，没有synchronized
     *
     * @return int
     */
    public int getValue() {
        return value;
    }

    /**
     * 写操作，synchronized 使其具备原子性
     */
    public synchronized void increment() {
        value++;
    }
}
```

适用于读操作频繁，写操作比较少的情况。如： 检测某一指标或者变量的值