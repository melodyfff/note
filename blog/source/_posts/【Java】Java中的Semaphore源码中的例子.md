---
title: 【Java】Java中的Semaphore源码中的例子
tags: [java]
date: 2019-7-22
---

# Java中的Semaphore源码中的例子
`Semaphore(信号量)`是`synchronized`的加强版，作用是控制线程的并发数量。

信号量通过使用计数器来控制对共享资源的访问。如果计数器大于零，则允许访问。如果为零，则拒绝访问。

计数器的计数是允许访问共享资源的许可。因此，要访问资源，必须从信号量向线程授予许可。 


[Semaphore Doc](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Semaphore.html) 中是这样定义的： 
```html
A counting semaphore. Conceptually, a semaphore maintains a set of permits. 

Each acquire() blocks if necessary until a permit is available, and then takes it. 

Each release() adds a permit, potentially releasing a blocking acquirer. 

However, no actual permit objects are used; the Semaphore just keeps a count of the number available and acts accordingly.

Semaphores are often used to restrict the number of threads than can access some (physical or logical) resource
```


## 例子

**@since 1.5**


```java
public class Pool {
    // 最大信号量
    private static final int MAX_AVAILABLE = 5;
    // 初始化 Semaphore
    private final Semaphore available = new Semaphore(MAX_AVAILABLE);

    // 获取数组中的数据，并且acquire()
    public Object getItem() throws InterruptedException {
        // 获取许可
        available.acquire();
        return getNextAvailableItem();
    }

    // 放回数据，并且release()
    public void putItem(Object x) {
        if (markAsUnUsed(x)){
            // 释放许可
            available.release();
        }
    }


    // Not a particularly efficient data structure; just for demo
    protected Object[] items = of(1, 2, 3, 4, 5);
    protected boolean[] used = new boolean[MAX_AVAILABLE];

    // 获取可用对象
    protected synchronized Object getNextAvailableItem() {
        for (int i = 0; i < MAX_AVAILABLE; ++i) {
            // 如果未被使用，设置为true表示已经被使用
            if (!used[i]) {
                used[i] = true;
                return items[i];
            }
        }
        // not reached
        return null;
    }


    // 释放许可
    protected synchronized boolean markAsUnUsed(Object item){
        for (int i = 0; i < MAX_AVAILABLE; ++i) {
            if (item == items[i]){
                // 如果已经被使用,设置为false表示释放
                if (used[i]){
                    used[i] = false;
                    return true;
                } else {
                    return false;
                }
            }
        }
        return false;
    }

    // 快速构造一个数组
    public static <T> T[] of(T... values) {
        return values;
    }

    public static void main(String[] args) throws InterruptedException {
        int N = 10;

        Pool pool = new Pool();
        
        // 模拟使用线程池
        for (int i = 0; i < N; i++) {
            // 由于permit大小设置为5,acquire()到第5个时
            // 如果未释放，线程将被阻塞
            if (i == 5){
                Thread.sleep(5000);
                System.out.println("Semaphore达到限制，无可用permit,需要release\n");

                // 释放permit ,这里可设置为 j > 0 ，少释放一个观察线程被阻塞的情况
                for (int j = i-1; j > -1; j--) {
                    pool.putItem(pool.items[j]);
                    System.out.println("release: " + pool.items[j]);
                }

                System.out.println();
            }


            System.out.println("acquire : " + pool.getItem());
        }
    }
}
```


结果为：

```bash
acquire : 1
acquire : 2
acquire : 3
acquire : 4
acquire : 5
Semaphore达到限制，无可用permit,需要release

release: 5
release: 4
release: 3
release: 2
release: 1

acquire : 1
acquire : 2
acquire : 3
acquire : 4
acquire : 5
```

## 参考

[Class Semaphore](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Semaphore.html)

[Semaphore in Java](https://www.geeksforgeeks.org/semaphore-in-java/)