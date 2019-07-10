---
title: 【Java】 volatile实现低开销的读写锁
tags: [java]
date: 2019-7-10
---

# volatile实现低开销的读写锁


- **volatile**关键字修饰后，能保证我们获取到的变量的值是最新的

- 只对写操作进行**synchronized**同步
- 适用于读操作频繁，写操作比较少的情况。如： 检测某一指标或者变量的值




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