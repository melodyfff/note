---
title: 【Java】多线程学习笔记
tags: [java]
date: 2016-9-22
---

Java多线程学习笔记

## 线程的生命周期及五种基本状态
![img1](../../../../img/1.jpg)  

![img2](../../../../img/2.jpg)
## Java线程具有五中基本状态
### 线程和进程一样分为五个阶段：创建、就绪、运行、阻塞、终止。
新建状态（New）：当线程对象对创建后，即进入了新建状态，如：Thread t = new MyThread();

就绪状态（Runnable）：当调用线程对象的start()方法（t.start();），线程即进入就绪状态。处于就绪状态的线程，只是说明此线程已经做好了准备，随时等待CPU调度执行，并不是说执行了t.start()此线程立即就会执行；

运行状态（Running）：当CPU开始调度处于就绪状态的线程时，此时线程才得以真正执行，即进入到运行状态。注：就     绪状态是进入到运行状态的唯一入口，也就是说，线程要想进入运行状态执行，首先必须处于就绪状态中；

阻塞状态（Blocked）：处于运行状态中的线程由于某种原因，暂时放弃对CPU的使用权，停止执行，此时进入阻塞状态，直到其进入到就绪状态，才 有机会再次被CPU调用以进入到运行状态。根据阻塞产生的原因不同，阻塞状态又可以分为三种：

1. 等待阻塞：运行状态中的线程执行wait()方法，使本线程进入到等待阻塞状态；

2. 同步阻塞 -- 线程在获取synchronized同步锁失败(因为锁被其它线程所占用)，它会进入同步阻塞状态；

3. 其他阻塞 -- 通过调用线程的sleep()或join()或发出了I/O请求时，线程会进入到阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。

死亡状态（Dead）：线程执行完了或者因异常退出了run()方法，该线程结束生命周期
## 进程与线程
<b>进程</b>：每个进程都有独立的代码和数据空间（进程上下文），进程间的切换会有较大的开销，一个进程包含1--n个线程。（进程是资源分配的最小单位）  

<b>线程</b>：同一类线程共享代码和数据空间，每个线程有独立的运行栈和程序计数器(PC)，线程切换开销小。（线程是cpu调度的最小单位）
## 实现多线程
### 继承Thread类（java.lang.Thread）
```java
/**
 * @Description:
 * @author xinchen
 * @date 2016年9月22日 下午10:41:25
 * @version V1.0
 */
class MyTread extends Thread {
    private String str;

    // 构造函数
    public MyTread(String str) {
        this.str = str;
    }

    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(str + "运行：" + i + "-------当前线程:" + Thread.currentThread().getName());
            try {
                sleep((int) Math.random() * 10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

public class Main {
    public static void main(String[] args) {
        MyTread mT1 = new MyTread("A");
        MyTread mT2 = new MyTread("B");
        System.out.println("当前线程："+Thread.currentThread().getName());
        mT1.start();
        mT2.start();
    }
}
```
运行结果：
```
当前线程：main
A运行：0-------当前线程:Thread-0
B运行：0-------当前线程:Thread-1
A运行：1-------当前线程:Thread-0
B运行：1-------当前线程:Thread-1
A运行：2-------当前线程:Thread-0
B运行：2-------当前线程:Thread-1
A运行：3-------当前线程:Thread-0
A运行：4-------当前线程:Thread-0
B运行：3-------当前线程:Thread-1
B运行：4-------当前线程:Thread-1
```
再运行一次
```
当前线程：main
A运行：0-------当前线程:Thread-0
A运行：1-------当前线程:Thread-0
A运行：2-------当前线程:Thread-0
B运行：0-------当前线程:Thread-1
A运行：3-------当前线程:Thread-0
B运行：1-------当前线程:Thread-1
A运行：4-------当前线程:Thread-0
B运行：2-------当前线程:Thread-1
B运行：3-------当前线程:Thread-1
B运行：4-------当前线程:Thread-1
```
### 结论：
- 当程序运行时，从`main()`进入，Java虚拟机启动一个进程，主线程`main`在`main()`中调用时被创建
随着调用两个`MyTread`的`star()`方法启动两个线程.  
- `start()`方法调用后不是立即执行`run()`中的代码，而是使该线程变为可运行态（Runnable）,什么时候
执行是由操作系统决定.  
- 从程序运行结果发现，线程是随机执行的。因此，<b>只有无序执行的代码才有必要设计多线程</b>
- 若`star()`重复，则抛出`java.lang.IllegalThreadStateException`异常.

### 实现Runnable接口（java.lang.Runnable）
```java
/**
 * @Description:
 * @author xinchen
 * @date 2016年9月22日 下午10:41:25
 * @version V1.0
 */
class MyTread implements Runnable {
    private String str;

    // 构造函数
    public MyTread(String str) {
        this.str = str;
    }

    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(str + "运行：" + i + "-------当前线程:" + Thread.currentThread().getName());
            try {
                Thread.sleep((int) Math.random() * 10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
public class Main {

    public static void main(String[] args) {
        MyTread mT1 = new MyTread("A");
        MyTread mT2 = new MyTread("B");
        Thread t1 = new Thread(mT1);
        Thread t2 = new Thread(mT2);
        System.out.println("当前线程："+Thread.currentThread().getName());
        t1.start();
        t2.start();
    }
}
```
运行结果：
```
当前线程：main
A运行：0-------当前线程:Thread-0
B运行：0-------当前线程:Thread-1
A运行：1-------当前线程:Thread-0
B运行：1-------当前线程:Thread-1
A运行：2-------当前线程:Thread-0
A运行：3-------当前线程:Thread-0
A运行：4-------当前线程:Thread-0
B运行：2-------当前线程:Thread-1
B运行：3-------当前线程:Thread-1
B运行：4-------当前线程:Thread-1
```
结论
- `MyThread`类通过实现`Runnable`接口，让该类拥有多线程的特征.
- `run()`是多线程中的一个约定,所有多线程执行代码都在里面.
- `Thread`类实际上也实现了`Runnable`接口.
- 在启动的多线程的时候，需要先通过`Thread`类的构造方法`Thread(Runnable target)` 构造出对象，
然后调用`Thread`对象的`start()`方法来运行多线程代码.
- `Runnable`定义的子类中没有`start()`方法，只有`Thread`类中才有,
也就是说可以通过Thread类来启动Runnable实现的多线程.

## Thread和Runnable的区别
- 如果一个类继承`Thread`，则不适合资源共享。但是如果实现了`Runable`接口的话，则很容易的实现资源共享.
- 增加程序的健壮性，代码可以被多个线程共享，代码和数据独立.
- 可以避免java中的单继承的限制.
- 线程池只能放入实现`Runable`或`callable`类线程，不能直接放入继承`Thread`的类.

### 通过Thread尝试实现资源共享
```java
class MyThread extends Thread {
    private int conut =5;   
    public void run(){  
        for(int i =1;i<10;i++){  
            if(this.conut>0){  
                System.out.println(Thread.currentThread().getName()+"输出---->"+(this.conut--));  
            }  
        }  
    }  
}
public class Main {
    public static void main(String[] args) {
        MyThread mT1 = new MyThread();
        MyThread mT2 = new MyThread();
        System.out.println("当前线程：" + Thread.currentThread().getName());
        mT1.start();
        mT2.start();
    }
}
```

结果：
```
当前线程：main
Thread-1输出---->5
Thread-1输出---->4
Thread-1输出---->3
Thread-1输出---->2
Thread-1输出---->1
Thread-0输出---->5
Thread-0输出---->4
Thread-0输出---->3
Thread-0输出---->2
Thread-0输出---->1
```

### 通过Runnable实现共享资源
```java
/**
 * @Description:
 * @author xinchen
 * @date 2016年9月22日 下午10:41:25
 * @version V1.0
 */
class MyThread implements Runnable {
    private int conut =5;   
    public void run(){  
        for(int i =1;i<5;i++){  
            if(this.conut>0){  
                System.out.println(Thread.currentThread().getName()+"输出---->"+(this.conut--));  
            }  
        }  
    }  
}
public class Main {
    public static void main(String[] args) {
        MyThread mT1 = new MyThread();
        System.out.println("当前线程：" + Thread.currentThread().getName());
        new Thread(mT1,"命名：线程1").start();
        new Thread(mT1,"命名：线程2").start();
    }
}
```
运行结果：
```
当前线程：main
命名：线程1输出---->5
命名：线程1输出---->4
命名：线程1输出---->3
命名：线程1输出---->2
命名：线程2输出---->1
```

结论：
- `main`方法其实也是一个线程。在`java`中所以的线程都是同时启动的，至于什么时候，哪个先执行，完全看谁先得到CPU的资源
- 在`java`中，每次程序运行至少启动2个线程。一个是`main`线程，一个是垃圾收集线程。因为每当使用java命令执行一个类的时候，实际上都会启动一个JVM，每一个JVM实现就是在操作系统中启动了一个进程

## 线程调度
### 调整线程优先级：  
线程具有优先级，优先级高的线程获得较多的运行机会  
`java`线程的优先级用整数表示,范围为1~10,`Thread`类有以下三个静态常量：  
```java
static int MAX_PRIORITY //线程具有的最高优先级，取值为10
static int MIN_PRIORITY //线程具有的最低优先级，取值为1
static int NORM_PRIORITY //分配给线程的默认优先级，取值为5
```
- `Thread`类的`setPriority()`和`getPriority()`方法来设置和获取线程的优先级
- 每个线程都有默认的优先级,主线程的默认优先级为`Thread.NORM_PRIORITY`
- 线程的优先级有继承关系,比如线程A中创建了线程B,那么B和A具有相同的优先级
- JVM提供了10个线程优先级,但是与常见的操作系统都不能很好的映射,如果希望程序能够移植到各个操作系统中,
应该仅仅使用`Thread`类有以下三个静态常量作为优先级,这样能保证同样的优先级采用了同样的调度方式

### 线程睡眠：
- `Thread.sleep(long millis)`方法，使线程转到阻塞状态
- `millis`参数设定睡眠的时间,以毫秒为单位
- 当睡眠结束后，就转为就绪`Runnable`状态
- `sleep()`具有良好平台移植性