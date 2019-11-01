---
title: 【Java】JUC - Monitor监控ThreadPoolExecutor
tags: [java]
date: 2019-11-1
---
# JUC - Monitor监控ThreadPoolExecutor
一个自定义`Monitor`监控ThreadPoolExecutor的执行情况

## TASK
**WokerTask**
```java
class WorkerTask implements Runnable{
        private String command;

        public WorkerTask(String command) {
            this.command = command;
        }

        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName()+" Start. Command = "+command);
            processCommand();
            System.out.println(Thread.currentThread().getName()+" End.");
        }

        private void processCommand(){
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }

        @Override
        public String toString() {
            return "WorkerTask{" +
                    "command='" + command + '\'' +
                    '}';
        }
    }
```

**MonitorTask(监听器)**
```java
class MonitorTask implements Runnable{
        // 被监控的executor
        private final ThreadPoolExecutor executor;
        // 监控间隔
        private final int seconds;
        // 监控开关
        private boolean run = true;

        public MonitorTask(ThreadPoolExecutor executor, int seconds) {
            this.executor = executor;
            this.seconds = seconds;
        }

        public void shutdown(){
            this.run = false;
        }
        @Override
        public void run() {
            while (run) {
                System.out.println(
                        String.format("%s - [monitor] [%d/%d] Active: %d, Completed: %d, Task: %d, Queue: %d, isShutdown: %s, isTerminated: %s",
                                Thread.currentThread().getName(),
                                this.executor.getPoolSize(),
                                this.executor.getCorePoolSize(),
                                this.executor.getActiveCount(),
                                this.executor.getCompletedTaskCount(),
                                this.executor.getTaskCount(),
                                this.executor.getQueue().size(),
                                this.executor.isShutdown(),
                                this.executor.isTerminated()));
                try {
                    TimeUnit.SECONDS.sleep(seconds);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

---

## RejectedExecutionHandler(拒绝策略)
**LogRejectedExecutionHandler**
```java
    class LogRejectedExecutionHandler implements RejectedExecutionHandler {
        @Override
        public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
            // 当任务队列满了,并且达到maximumPoolSize时的拒绝策略
            System.out.println(r.toString() + " is rejected");
        }
    }
```
---

## ThreadPoolExecutor
```java
ThreadPoolExecutor poolExecutor = new ThreadPoolExecutor(
                2,
                4,
                10, TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(2),
                Executors.defaultThreadFactory(),
                new LogRejectedExecutionHandler()
        );
```

---

## 完整的例子
```java
public class ThreadPoolExecutorMonitorSimple {
    public static void main(String[] args) throws InterruptedException {
        final ThreadPoolExecutor poolExecutor = new ThreadPoolExecutor(
                2,
                4,
                10, TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(2),
                Executors.defaultThreadFactory(),
                new LogRejectedExecutionHandler()
        );

        // 创建监控任务,间隔3s
        final MonitorTask monitorTask = new MonitorTask(poolExecutor, 3);

        // 监听任务启动
        new Thread(monitorTask).start();

        for (int i = 0; i < 10; i++) {
            poolExecutor.execute(new WorkerTask("cmd-"+i));
        }

        TimeUnit.SECONDS.sleep(60);

        poolExecutor.shutdown();

        TimeUnit.SECONDS.sleep(5);

        monitorTask.shutdown();
    }

    static class LogRejectedExecutionHandler implements RejectedExecutionHandler {
        @Override
        public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
            // 当任务队列满了,并且达到maximumPoolSize时的拒绝策略
            System.out.println(r.toString() + " is rejected");
        }
    }

    static class WorkerTask implements Runnable{
        private String command;

        public WorkerTask(String command) {
            this.command = command;
        }

        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName()+" Start. Command = "+command);
            processCommand();
            System.out.println(Thread.currentThread().getName()+" End.");
        }

        private void processCommand(){
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }

        @Override
        public String toString() {
            return "WorkerTask{" +
                    "command='" + command + '\'' +
                    '}';
        }
    }

    static class MonitorTask implements Runnable{
        // 被监控的executor
        private final ThreadPoolExecutor executor;
        // 监控间隔
        private final int seconds;
        // 监控开关
        private boolean run = true;

        public MonitorTask(ThreadPoolExecutor executor, int seconds) {
            this.executor = executor;
            this.seconds = seconds;
        }

        public void shutdown(){
            this.run = false;
        }
        @Override
        public void run() {
            while (run) {
                System.out.println(
                        String.format("%s - [monitor] [%d/%d] Active: %d, Completed: %d, Task: %d, Queue: %d, isShutdown: %s, isTerminated: %s",
                                Thread.currentThread().getName(),
                                this.executor.getPoolSize(),
                                this.executor.getCorePoolSize(),
                                this.executor.getActiveCount(),
                                this.executor.getCompletedTaskCount(),
                                this.executor.getTaskCount(),
                                this.executor.getQueue().size(),
                                this.executor.isShutdown(),
                                this.executor.isTerminated()));
                try {
                    TimeUnit.SECONDS.sleep(seconds);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

**日志:**
```bash
pool-1-thread-1 Start. Command = cmd-0
pool-1-thread-2 Start. Command = cmd-1
pool-1-thread-3 Start. Command = cmd-4
WorkerTask{command='cmd-6'} is rejected
WorkerTask{command='cmd-7'} is rejected
WorkerTask{command='cmd-8'} is rejected
WorkerTask{command='cmd-9'} is rejected
pool-1-thread-4 Start. Command = cmd-5
Thread-0 - [monitor] [0/2] Active: 0, Completed: 0, Task: 1, Queue: 0, isShutdown: false, isTerminated: false
Thread-0 - [monitor] [4/2] Active: 4, Completed: 0, Task: 6, Queue: 2, isShutdown: false, isTerminated: false
pool-1-thread-1 End.
pool-1-thread-1 Start. Command = cmd-2
pool-1-thread-2 End.
pool-1-thread-2 Start. Command = cmd-3
pool-1-thread-3 End.
pool-1-thread-4 End.
Thread-0 - [monitor] [4/2] Active: 2, Completed: 4, Task: 6, Queue: 0, isShutdown: false, isTerminated: false
Thread-0 - [monitor] [4/2] Active: 2, Completed: 4, Task: 6, Queue: 0, isShutdown: false, isTerminated: false
pool-1-thread-1 End.
pool-1-thread-2 End.
Thread-0 - [monitor] [4/2] Active: 0, Completed: 6, Task: 6, Queue: 0, isShutdown: false, isTerminated: false
Thread-0 - [monitor] [2/2] Active: 0, Completed: 6, Task: 6, Queue: 0, isShutdown: false, isTerminated: false
Thread-0 - [monitor] [2/2] Active: 0, Completed: 6, Task: 6, Queue: 0, isShutdown: false, isTerminated: false
Thread-0 - [monitor] [2/2] Active: 0, Completed: 6, Task: 6, Queue: 0, isShutdown: false, isTerminated: false
Thread-0 - [monitor] [2/2] Active: 0, Completed: 6, Task: 6, Queue: 0, isShutdown: false, isTerminated: false
Thread-0 - [monitor] [2/2] Active: 0, Completed: 6, Task: 6, Queue: 0, isShutdown: false, isTerminated: false
Thread-0 - [monitor] [2/2] Active: 0, Completed: 6, Task: 6, Queue: 0, isShutdown: false, isTerminated: false
Thread-0 - [monitor] [2/2] Active: 0, Completed: 6, Task: 6, Queue: 0, isShutdown: false, isTerminated: false
Thread-0 - [monitor] [2/2] Active: 0, Completed: 6, Task: 6, Queue: 0, isShutdown: false, isTerminated: false
Thread-0 - [monitor] [2/2] Active: 0, Completed: 6, Task: 6, Queue: 0, isShutdown: false, isTerminated: false
Thread-0 - [monitor] [2/2] Active: 0, Completed: 6, Task: 6, Queue: 0, isShutdown: false, isTerminated: false
Thread-0 - [monitor] [2/2] Active: 0, Completed: 6, Task: 6, Queue: 0, isShutdown: false, isTerminated: false
Thread-0 - [monitor] [2/2] Active: 0, Completed: 6, Task: 6, Queue: 0, isShutdown: false, isTerminated: false
Thread-0 - [monitor] [2/2] Active: 0, Completed: 6, Task: 6, Queue: 0, isShutdown: false, isTerminated: false
Thread-0 - [monitor] [2/2] Active: 0, Completed: 6, Task: 6, Queue: 0, isShutdown: false, isTerminated: false
Thread-0 - [monitor] [2/2] Active: 0, Completed: 6, Task: 6, Queue: 0, isShutdown: false, isTerminated: false
Thread-0 - [monitor] [0/2] Active: 0, Completed: 6, Task: 6, Queue: 0, isShutdown: true, isTerminated: true
Thread-0 - [monitor] [0/2] Active: 0, Completed: 6, Task: 6, Queue: 0, isShutdown: true, isTerminated: true
```

---

## 参考阅读
- [Java Thread Pool – ThreadPoolExecutor Example](https://howtodoinjava.com/java/multi-threading/java-thread-pool-executor-example/)
- [ThreadPoolExecutor – Java Thread Pool Example](https://www.journaldev.com/1069/threadpoolexecutor-java-thread-pool-example-executorservice)
- [Java 多线程（5）：Fork/Join 型线程池与 Work-Stealing 算法](https://segmentfault.com/a/1190000008140126)