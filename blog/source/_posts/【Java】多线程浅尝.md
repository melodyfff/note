---
title: 【Java】多线程浅尝
tags: [java]
date: 2019-3-17
---

# 多线程浅尝

> 参考： http://wideskills.com/java-tutorial/java-threads-tutorial  
> 关于`volatile`参考：https://blog.csdn.net/u010715440/article/details/79503505
## 线程的创建

Java提供三种创建线程的方式：

- 通过实现`Runnable`接口
- 通过继承`Thread`类
- 通过实现`Callable`接口和`Future`创建

## Callable创建例子
```java
public class CallableTest implements Callable<Integer> {

    @Override
    public Integer call() throws Exception {
        int i = 0;
        for (;i<100;i++){
            System.out.println(Thread.currentThread().getName()+": "+i);
        }
        return i;
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CallableTest ctt = new CallableTest();
        FutureTask<Integer> callBack = new FutureTask<>(ctt);
        new Thread(callBack,"Thread1").start();
        System.out.println(Thread.currentThread().getName()+": "+callBack.get());
    }
}
```



## 线程的同步(synchronization)

- `synchronized methods`
- `synchronized blocks`


### synchronized methods

```java
public class SyncMethodsExample extends Thread {
    static Integer[] msg = {1, 2, 3, 4, 5};

    private Random random = ThreadLocalRandom.current();

    public SyncMethodsExample(String name) {
        super(name);
    }

    /**
     * random wait
     */
    void randomWait() {
        try {
            sleep((long) (3000 * random.nextDouble()));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void run() {
        SynchronizedOutput.displayList(getName(),msg);
    }


    public static void main(String[] args) {
        SyncMethodsExample t1 = new SyncMethodsExample("thread1");
        SyncMethodsExample t2 = new SyncMethodsExample("thread2");
        t1.start();
        t2.start();

        boolean t1IsAlive = true;
        boolean t2IsAlive = true;

        do {
            if (t1IsAlive && !t1.isAlive()){
                t1IsAlive = false;
                System.out.println(t1.getName()+" is dead");
            }

            if (t2IsAlive && !t2.isAlive()){
                t2IsAlive = false;
                System.out.println(t2.getName()+" is dead");
            }
        } while (t1IsAlive || t2IsAlive);

        System.out.println(Thread.currentThread().getName()+": done");
    }

}

class SynchronizedOutput {
    // if the 'synchronized' keyword is removed, the message is displayed in random fashion
    public static synchronized void displayList(String name, Integer[] msg) {
        for (int i = 0; i < msg.length; i++) {
            SyncMethodsExample t = (SyncMethodsExample) Thread.currentThread();
            t.randomWait();
            System.out.println(name + ": " + msg[i]);
        }
    }
}
```

Result:
```bash
thread1: 1
thread1: 2
thread1: 3
thread1: 4
thread1: 5
thread1 is dead
thread2: 1
thread2: 2
thread2: 3
thread2: 4
thread2: 5
thread2 is dead
main: done
```

### synchronized blocks
```java
public class SyncBlockExample extends Thread {
    static Integer[] msg = {1, 2, 3, 4, 5};
    private final Random random = ThreadLocalRandom.current();

    public SyncBlockExample(String name) {
        super(name);
    }

    void randomWait() {
        try {
            sleep((long) (300 * random.nextDouble()));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void run() {
        synchronized (random) {
            for (int i = 0; i < msg.length; i++) {
                randomWait();
                System.out.println(getName() + ": " + msg[i]);
            }
        }
    }

    public static void main(String[] args) {
        SyncBlockExample t1 = new SyncBlockExample("thread1");
        SyncBlockExample t2 = new SyncBlockExample("thread2");
        t1.start();
        t2.start();

        boolean t1IsAlive = true;
        boolean t2IsAlive = true;
        do {
            if (t1IsAlive && !t1.isAlive()) {
                t1IsAlive = false;
                System.out.println(t1.getName() + " is dead");
            }

            if (t2IsAlive && !t2.isAlive()) {
                t2IsAlive = false;
                System.out.println(t2.getName() + " is dead");
            }
        } while (t1IsAlive || t2IsAlive);
        System.out.println(Thread.currentThread().getName() + ": done");
    }
}
```

Result:
```bash
thread1: 1
thread1: 2
thread1: 3
thread1: 4
thread1: 5
thread1 is dead
thread2: 1
thread2: 2
thread2: 3
thread2: 4
thread2: 5
thread2 is dead
main: done
```

In summary, a thread can hold a lock on an object

- by executing a synchronized instance method of the object
- by executing the body of a synchronized block that synchronizes on the object
- by executing a synchronized static method of a class

## 线程的等待和唤醒(WAITING AND NOTIFYING)
下面的程序显示了三个线程，操作相同的堆栈。 其中两个是在堆栈上推送元素，而第三个是从堆栈中弹出元素。 此示例说明了作为调用对象上的`wait()`方法的结果等待的线程如何被另一个调用同一对象上的`notify()`方法的线程通知


****StackClass:****
```java
public class StackClass {

    private Object[] stackArray;
    private volatile int topOfStack;

    public StackClass(int capacity) {
        stackArray = new Object[capacity];
        topOfStack = 1;
    }

    public synchronized Object pop() {
        System.out.println(Thread.currentThread() + ": popping");
        while (isEmpty()) {
            try {
                System.out.println(Thread.currentThread() + ": waiting to pop");
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        Object obj = stackArray[topOfStack];
        stackArray[topOfStack--] = null;
        System.out.println(Thread.currentThread() + ": notifying after pop");
        notify();
        return obj;
    }

    public synchronized void push(Object element) {
        System.out.println(Thread.currentThread() + ": pushing");
        while (isFull()) {
            try {
                System.out.println(Thread.currentThread() + ": topOfStack is " + topOfStack);
                System.out.println(Thread.currentThread() + ": waiting to push");
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        stackArray[++topOfStack] = element;
        notify();
        System.out.println(Thread.currentThread() + ": notifying after push");
    }

    /**
     * judge stackArray is full
     * @return boolean
     */
    public boolean isFull() {
        return topOfStack >= stackArray.length - 1;
    }
    /**
     * judge stackArray is empty
     * @return boolean
     */
    public boolean isEmpty() {
        return topOfStack < 0;
    }
}
```

****StackUser:****
```java
public abstract class StackUser extends Thread{
    protected StackClass stack;

    public StackUser(String name, StackClass stack) {
        super(name);
        this.stack = stack;
        System.out.println(this);
        setDaemon(true);
        start();
    }
}
```

****StackPusher：****
```java
public class StackPusher extends StackUser{

    public StackPusher(String name, StackClass stack) {
        super(name, stack);
    }

    @Override
    public void run() {
        while (true){
            stack.push(1);
        }
    }
}
```

****StackPopper:****
```java
public class StackPopper extends StackUser{

    public StackPopper(String name, StackClass stack) {
        super(name, stack);
    }

    @Override
    public void run() {
        while (true){
            stack.pop();
        }
    }
}
```

****WaitAndNotifyExample:****
```java
public class WaitAndNotifyExample {
    public static void main(String[] args) {
        StackClass stack = new StackClass(5);
        new StackPusher("Pusher1", stack);
        new StackPusher("Pusher2", stack);
        new StackPopper("Popper1", stack);
        System.out.println("Main Thread sleeping.");

        try {
            Thread.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Exit from Main Thread.");
    }
}
```



类`StackClass`中的字段`topOfStack`被声明为`volatile`，因此对此变量的读写操作将在运行时访问此变量的主值，而不是任何副本。

由于线程操作相同的堆栈对象并且类`StackClass`中的`push()`和`pop(0`方法同步，这意味着线程在同一对象上同步

程序如何使用`wait()`和`notify()`进行线程间通信?

- `synchronized pop()`方法 - 当在`StackClass`对象上执行此方法的线程发现堆栈为空时，它调用`wait()`方法以等待其他一些线程通过使用`synchronized`来填充堆栈推。一旦其他线程进行推送，它就会调用`notify`方法。

- `synchronized push()`方法 - 当在`StackClass`对象上执行此方法的线程发现堆栈已满时，它调用`wait()`方法等待一些其他线程移除一个元素，为新的被推元素。
一旦另一个线程弹出，它就会调用`notify`方法。

## 线程的加入(JOINING)

线程在另一个线程上调用`join()`方法，以便等待另一个线程完成其执行。
考虑一个线程t1在线程t2上调用方法`join()`。 如果线程t2已经完成，则`join()`调用无效。 如果线程t2仍然存在，则线程t1转换到阻塞加入完成状态。

下面是一个程序，显示了线程如何调用重载的线程连接方法。

```java
public class ThreadJoinDemo {
    public static void main(String[] args) {
        Thread t1 = new JoinThread("T1");
        Thread t2 = new JoinThread("T2");

        try {
            System.out.println("Wait for thre child threads to finish.");

            t1.join();
            if (!t1.isAlive()){
                System.out.println("Thread T1 is not alive.");
            }

            t2.join();
            if (!t2.isAlive()){
                System.out.println("Thread T2 is not alive.");
            }

        } catch (InterruptedException e){
            e.printStackTrace();
        }

        System.out.println("Exit from Main Thread.");
    }
}


class JoinThread extends Thread {

    private final Random random = ThreadLocalRandom.current();

    public JoinThread(String name) {
        super(name);
        start();
    }

    void randWait() {
        try {
            sleep((long) (3000 * random.nextDouble()));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }


    @Override
    public void run() {
        System.out.println(getName() + ": is running.");
        for (int i = 0; i < 3; i++) {
            System.out.println(getName() + ": " + i);
            randWait();
        }
    }
}
```

****Result:****
```bash
Wait for thre child threads to finish.
T1: is running.
T2: is running.
T1: 0
T2: 0
T2: 1
T1: 1
T1: 2
T2: 2
Thread T1 is not alive.
Thread T2 is not alive.
Exit from Main Thread.
```

## 多线程的死锁(DEADLOCK)

在每个线程等待无法使用的资源时，程序会出现死锁的情况。 最简单的死锁形式是当两个线程都在等待另一个线程锁定的资源时。 由于每个线程都在等待另一个线程放弃锁定，因此它们都会在`Blocked-for-lock-acquisition`状态下永远等待。 

线程t1尝试首先在字符串o1上同步，然后在字符串o2上同步。 线程t2正好相反。 它首先在字符串o2上同步，然后在字符串o1上同步。 因此，如上所述，可能发生死锁。

下面是一个程序，它说明了多线程应用程序中的死锁
```java
public class DeadLockExample {

    String o1 = "Lock";
    String o2 = "Step";

    Thread t1 = (new Thread("Printer1") {
        @Override
        public void run() {
            while (true){
                synchronized (o1){
                    synchronized (o2){
                        System.out.println(getName()+ ": " + o1 + o2);
                    }
                }
            }
        }
    });

    Thread t2 = (new Thread("Printer2") {
        @Override
        public void run() {
            while (true){
                synchronized (o2){
                    synchronized (o1){
                        System.out.println(getName()+ ": " + o2 + o1);
                    }
                }
            }
        }
    });

    public static void main(String[] args) {
        DeadLockExample lock = new DeadLockExample();
        lock.t1.start();
        lock.t2.start();
    }
}
```