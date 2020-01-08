---
title: 【Java】JUC - 一把Synchronized的简单无用锁
tags: [java]
date: 2020-1-08
---
# JUC - 一把Synchronized的简单无用锁

想自己实现一把简单的锁，基于`Synchronized`,可以和`ReentrantLock`一样可以重入


**UselessLock.java**
```java
public class UselessLock implements Lock {

    private volatile int state = 0;

    @Override
    public void lock() {
        // 对当前UselessLock对象进行加锁
        synchronized (this) {
            // 当n != 0 的时候说明已经被其他线程持有
            while (state != 0) {
                try {
                    this.wait(); // CAS自旋也行，这里wait()直接线程直接进入 waiting 等待被唤醒
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            // 上锁
            state = 1;
        }
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {}

    @Override
    public boolean tryLock() {return false;}

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {return false;}

    @Override
    public void unlock() {
        synchronized (this) {
            // 释放锁
            state = 0;
            this.notifyAll(); // 唤醒等待this对象的所有线程
        }
    }

    @Override
    public Condition newCondition() {return null;}
}
```

使用测试

```java
    public static void main(String[] args) throws InterruptedException {
        // 构造一个不可变地址
        final int[] m = {0};

        // 使用自定义锁
        Lock lock = new UselessLock();

        // 线程组
        Thread[] threads = new Thread[100];

        Runnable task = () -> {
            for (int i = 0; i < 100; i++) {
                lock.lock(); // 上锁
                try {
                    m[0]++; // m += 1
                } finally {
                    lock.unlock(); // 解锁
                }
            }
        };

        // 顺序启动
        for (int i = 0; i < 100; i++) {
            threads[i] = new Thread(task);
            threads[i].start();
        }

        // 等待所有线程完成
        for (int i = 0; i < 100; i++) {
            threads[i].join();
        }

        // 输出最终结果
        System.out.println(m[0]);
    }
```

## 关于synchronized(this)和synchronized(Class)

上面的例子是用的`Synchronized(this)`,对象的监视器是针对实例对象本身

一个简单`synchronized(Class)`例子
```java
public class UselessLock implements Lock {

    private static volatile int state = 0;

    @Override
    public void lock() {
        synchronized (UselessLock.class) {
            // 当n != 0 的时候说明已经被其他线程持有
            while (state != 0) {
                try {
                    UselessLock.class.wait(); // CAS自旋也行，这里wait()直接线程直接进入 waiting 等待被唤醒
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            // 上锁
            state = 1;
        }
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {

    }

    @Override
    public boolean tryLock() {
        return false;
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return false;
    }

    @Override
    public void unlock() {
        synchronized (UselessLock.class) {
            // 释放锁
            state = 0;
            UselessLock.class.notifyAll(); // 唤醒等待this对象的所有线程
        }
    }
    @Override
    public Condition newCondition() {
        return null;
    }
}
```