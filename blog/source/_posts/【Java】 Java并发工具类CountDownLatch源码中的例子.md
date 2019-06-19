---
title: 【Java】 【Java】 Java并发工具类CountDownLatch源码中的例子
tags: [java]
date: 2019-6-19
---

# Java并发工具类CountDownLatch源码中的例子

## 实例一

#### 原文描述
```java
/**
 * <p><b>Sample usage:</b> Here is a pair of classes in which a group
 * of worker threads use two countdown latches:
 * <ul>
 * <li>The first is a start signal that prevents any worker from proceeding
 * until the driver is ready for them to proceed;
 * <li>The second is a completion signal that allows the driver to wait
 * until all workers have completed.
 * </ul>
 **/
```

样本用法：这是一对组中的一个类工作线程使用两个倒计时锁存器：

第一个是启动信号，阻止任何工作人员继续进行直到司机准备好继续进行;

第二个是完成信号，允许驾驶员等待直到所有工人完成。

**实例代码**
```java
public class Driver {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch startSignal = new CountDownLatch(1);
        CountDownLatch doneSignal = new CountDownLatch(5);

        for (int i=0;i<5;++i){
            new Thread(new Worker(startSignal, doneSignal)).start();
        }

        System.out.println("ALL THREAD START UP.");

        // 任务开始信号
        startSignal.countDown();

        System.out.println(" startSignal count down. ");

        // 等待所有线程执行完成后才执行后续代码
        doneSignal.await();

        System.out.println(" ALL THREAD END UP. ");


        // 执行结果
        // ALL THREAD START UP.
        // startSignal count down. 
        // Hello World!
        // Hello World!
        // Hello World!
        // Hello World!
        // Hello World!
        // ALL THREAD END UP.
    }
}

class Worker implements Runnable{

    private final CountDownLatch startSignal;
    private final CountDownLatch doneSignal;

    Worker(CountDownLatch startSignal, CountDownLatch doneSignal) {
        this.startSignal = startSignal;
        this.doneSignal = doneSignal;
    }

    @Override
    public void run() {
        try {
            // 阻止线程执行，直到startSignal.countDown()后开始执行
            startSignal.await();
            doWork();
            doneSignal.countDown();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }

    void doWork(){
        System.out.println("Hello World!");
    }
}
```

## 实例二

#### 原文描述
```java
/*
 * <p>Another typical usage would be to divide a problem into N parts,
 * describe each part with a Runnable that executes that portion and
 * counts down on the latch, and queue all the Runnables to an
 * Executor.  When all sub-parts are complete, the coordinating thread
 * will be able to pass through await. (When threads must repeatedly
 * count down in this way, instead use a {@link CyclicBarrier}.)
 **/
```

另一种典型用法是将问题分成N个部分，用执行该部分的`Runnable`描述每个部分倒计时锁定，并将所有`Runnables`排队到执行人。 

当所有子部件完成时，协调线程将能够通过等待。(当线程必须重复时以这种方式倒数，而不是使用`{@link CyclicBarrier}`)

**实例代码**
```java
public class Driver2 {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch doneSignal = new CountDownLatch(5);

        // 创建线程池
        Executor executor = Executors.newSingleThreadExecutor();

        for (int i = 0;i<5;++i){
            // create and start threads
            executor.execute(new WorkerRunnable(doneSignal,i));
        }

        // 等待所有线程任务执行完成
        doneSignal.await();

        System.out.println("EXECUTOR DOWN.");

        // 关闭线程池
        ((ExecutorService) executor).shutdown();


        // 输出结果
        //  Hello World! : 0
        //  Hello World! : 1
        //  Hello World! : 2
        //  Hello World! : 3
        //  Hello World! : 4
        //  EXECUTOR DOWN.
    }
}

class WorkerRunnable implements Runnable {

    private final CountDownLatch doneSignal;
    private final int i;

    WorkerRunnable(CountDownLatch doneSignal, int i) {
        this.doneSignal = doneSignal;
        this.i = i;
    }

    @Override
    public void run() {
        doWork(i);
        doneSignal.countDown();
    }

    void doWork(int count) {
        System.out.println("Hello World! : " + count);
    }
}
```