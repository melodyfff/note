---
title: 【Java】JUC - Condition
tags: [java]
date: 2020-1-09
---
# JUC - Condition

## 简介
`Conditions`（也称为`条件队列(condition queues)`或 `条件变量 (condition variables)`）为一个线程`暂停执行（"to wait"）`直到被另一个线程`notified`，提供一个可判断`条件状态（state condition）`的方法. 

由于对该共享状态信息的访问发生在不同的线程中，因此必须对其进行保护，因此某种形式的锁与该条件(condition)总是相关联出现.等待条件提供的关键是，它自动释放关联的锁并挂起当前线程，就像`Object.wait`

一个`Condition`实例本质上绑定到`锁Lock`。要在特定`Lock的实例`中获取`Condition实例` ，请使用其`newCondition()`方法。
```java
lock.newCondition()
```

## 例子
假设我们有一个支持`put`和`take`方法的有限缓冲区 。如果`take`尝试在空缓冲区上调用 ，则线程将阻塞，直到有可用项为止。如果put尝试在满缓冲区上调用，则线程将阻塞，直到有可用空间为止。

我们希望将等待put线程和take线程保持在单独的等待集(wait-sets)中，这样我们就可以使用仅在缓冲区中的项目或空间可用时才通知单个线程的优化。这可以使用两个`Condition`实例来实现 。

```java
public class BoundedBuffer {
    final Lock lock = new ReentrantLock();
    final Condition notFull = lock.newCondition();
    final Condition notEmpty = lock.newCondition();

    final Object[] items = new Object[100];
    int putptr,takeptr, count;

    public void put(Object x) throws InterruptedException {
        lock.lock();
        try {
            // 当队列容量达到上限时
            while (count == items.length){
                // Condition notFull不成立, park当前线程
                // 直到notFull.signal() 释放信号唤醒在此条件下进入wait-sets的线程
                notFull.await();
            }

            items[putptr] = x;

            // 循环放值,当达到最大上限,回归至初始位置
            if (++putptr == items.length){
                putptr = 0;
            }

            // 计数
            ++count;

            // Condition notEmpty条件成立(至少刚刚放入了一个),
            // 发送信号唤醒因为notEmpty进入wait-sets的线程
            notEmpty.signal();
        }finally {
            lock.unlock();
        }
    }

    public Object take() throws InterruptedException {
        lock.lock();
        try {
            // 当队列中为空,并且尝试取值
            while (count == 0){
                // Condition notEmpty不成立, 直接park线程,
                // 直至notEmpty.signal() 释放信号唤醒此条件下进入wait-sets的线程
                notEmpty.await();
            }

            // 取值
            Object x = items[takeptr];

            // 循环取值,当达到最大长度的时候,回归至初始位置
            if (++takeptr == items.length){
                takeptr = 0;
            }

            // 记录当前容纳的数目
            --count;

            // 发送信号, Condition notFull成立(因为最少刚刚消费了一个)
            // 唤醒因为notFull条件进入wait-sets的线程
            notFull.signal();
            return x;
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        final BoundedBuffer boundedBuffer = new BoundedBuffer();
        Runnable taks = () -> {
            try {
                boundedBuffer.put("ok");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        };

        // 塞满队列,当101次塞入时,进入等待
        for (int i = 0; i < 101; i++) {
            new Thread(taks).start();
        }

        TimeUnit.SECONDS.sleep(3);

        // 消耗一次,唤醒之前塞入队列时进入等待的线程
        boundedBuffer.take();
    }
}
```

## 参考
[oracle javase 8 doc - Condition](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/Condition.html)
