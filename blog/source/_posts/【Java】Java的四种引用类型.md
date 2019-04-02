---
title: 【Java】 【Java】Java的四种引用类型
tags: [Java]
date: 2019-4-2
---

# Java的四种引用类型

## 历史回顾

### `JDK1.2`之前

如果`reference`类型的数据中存储的数值代表的是另一块内存的起始地址，就称为这块内存代表着一个引用

`Java`中的垃圾回收机制在判断是否回收某个对象的时候，都需要依据`引用`这个概念。

在不同的垃圾回收算法中，对引用的判断方式有所不同：

- **引用计数法：** 为每个对象添加一个引用计数器，每当有一个引用指向它的时候，计数器就+1,当引用失效时，计数器就-1,当计数器为0时，则认为该对象可以回收(目前Java中已经弃用这种方式)

- **可达性分析算法:** 从名为`GC ROOTS`的对象开始向下搜索，如果一个对象到`GC ROOTS`没有任何引用链相连时，则说明此对象不可用。

`JDK1.2`之前，一个对象只有`已被引用`和`未被引用`两种状态，这将无法描述某些特殊情况下的对象，比如：当内存充足的时候需要保留，而内存紧张时才需要被抛弃的一类对象。


### `JDK1.2`之后

`Java`对引用的概念进行了扩充，将引用分为以下四种（强度依次减弱）：

- **强引用(Strong Reference)**
- **软引用(Soft Reference)**
- **弱引用(Weak Reference)**
- **虚引用(Phantom Reference)**

以上四种引用都继承于`java.lang.ref.Reference`

## 强引用(FinalReference / Finalizer)

`Java`中的默认声明就是强引用，如:
```java
// 只要 o 还指向 Object对象，Object对象就不会被回收
Object o = new Object();

// 设置为null后可被JVM回收
o = null;
```

强引用可以直接访问引用对象，而且强引用只在存在，那么引用对象就不会被JVM垃圾回收，

即使JVM抛出OOM异常的时候，也不回收强引用引用的对象，所以不正确的使用方式可能会造成内存泄漏。

如果想中断强引用与对象之间的联系，可以显示的将强引用赋值为null，这样一来，JVM就可以适时的回收对象了

`java.lang.ref.FinalReference`和`java.lang.ref.Finalizer`均为包级别访问范围

下面加入vm参数 `-XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xmx5m` 将最大可用内存设置为5M

```java
   public static void main(String[] args) {
        // 通过声明的方式生成一个强引用
        Object object = new Object();

        // java.lang.Object@2b193f2d
        System.out.println(object);

        // 执行gc
        System.gc();

        // 执行gc后，强引用不会被回收
        // java.lang.Object@2b193f2d
        System.out.println(object);

        // 分配一块5M内存，此时JVM内存耗尽，抛出OOM错误(java.lang.OutOfMemoryError: Java heap space)
        try {
            byte[] buf = new byte[5 * 1024 * 1024];
        } catch (Throwable throwable){
            System.out.println(throwable);
        }

        // 即使内存溢出，也不回收强引用对象
        // java.lang.Object@2b193f2d
        System.out.println(object);
    }
```

## 软引用(SoftReference)

软引用是用来描述一些非必需但仍有用的对象。

在内存足够的时候，软引用对象不会被回收，只有在内存不足时，系统则会回收软引用对象，如果回收了软引用对象之后仍然没有足够的内存，才会抛出内存溢出异常

这种特性常常被用来实现缓存技术，比如网页缓存，图片缓存等。

软引用可以和一个引用队列（`ReferenceQueue`）联合使用，如果软引用所引用的对象被垃圾回收器回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中。

在 `JDK1.2` 之后，用`java.lang.ref.SoftReference`类来表示软引用。

下面我们加入VM参数`-XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xms2M -Xmx3M`，将初始内存设置为2M，最大内存设置为3M

```java
public class OOM {
    private static List<Object> list = new ArrayList<>();

    public static void main(String[] args) {
        testSoftReference();
    }

    private static void testSoftReference() {
        // 创建10个1M大小的字节数组，赋值给软引用，此时-Xms3m
        for (int i = 0; i < 10; i++) {
            byte[] buff = new byte[1 * 1024 * 1024];
            SoftReference<byte[]> softReference = new SoftReference<>(buff);
            list.add(softReference);
        }

        // 通知进行gc
        System.gc();

        // 循环变量输出
        for (int i = 0;i<10;i++){
            Object obj = ((SoftReference) list.get(i)).get();
            System.out.println(obj);
        }
    }
}
```

输出结果为:
```bash
null
null
null
null
null
null
null
null
null
[B@2b193f2d
```

无论循环创建多少个软引用对象，打印结果总是只有最后一个对象被保留，其他的obj全都被置空回收了。

这里就说明了在内存不足的情况下，软引用将会被自动回收。

值得注意的一点 , 即使有 byte[] buff 引用指向对象, 且 buff 是一个`strong reference`, 但是 `softReference` 指向的对象仍然被回收了，这是因为Java的编译器发现了在之后的代码中, buff 已经没有被使用了, 所以自动进行了优化。

如果稍微修改下
```java
private static void testSoftReference() {
        byte[] buff = null;
        // 创建10个1M大小的字节数组，赋值给软引用，此时-Xms3m
        for (int i = 0; i < 10; i++) {
            buff = new byte[1 * 1024 * 1024];
            SoftReference<byte[]> softReference = new SoftReference<>(buff);
            list.add(softReference);
        }

        // 通知进行gc
        System.gc();

        // 循环变量输出
        for (int i = 0;i<10;i++){
            Object obj = ((SoftReference) list.get(i)).get();
            System.out.println(obj);
        }
    }
```

buff会因为强引用的存在，无法被垃圾回收，抛出OOM错误

如果一个对象惟一剩下的引用是软引用，那么该对象是软可及的`softly reachable`

垃圾收集器并不像其收集弱可及的对象一样尽量地收集软可及的对象，相反，它只在真正 `需要` 内存时才收集软可及的对象。

## 弱引用(WeakReference)

弱引用的引用强度比软引用要更弱一些

无论内存是否足够，只要 JVM 开始进行垃圾回收，那些被弱引用关联的对象都会被回收

在 `JDK1.2` 之后，用 `java.lang.ref.WeakReference` 来表示弱引用。

使用和测试软引用的方式来测试弱应用
```java
    private static void testWeakReference() {
        // 创建10个1M大小的字节数组，赋值给弱引用，此时-Xms3m
        for (int i = 0; i < 10; i++) {
            byte[] buff = new byte[1 * 1024 * 1024];
            WeakReference<byte[]> softReference = new WeakReference<>(buff);
            list.add(softReference);
        }

        // 通知进行gc
        System.gc();

        // 循环变量输出
        for (int i = 0;i<10;i++){
            Object obj = ((WeakReference) list.get(i)).get();
            System.out.println(obj);
        }
    }
```

输出结果为
```bash
null
null
null
null
null
null
null
null
null
null
```

## 虚引用(PhantomReference)

虚引用是最弱的一种引用关系

虚引用和前面的软引用、弱引用不同，它并不影响对象的生命周期，这种引用引用的对象随时都可能被回收，而且它的get方法永远返回null

也就是说将永远无法通过虚引用来获取对象，虚引用必须要和 `ReferenceQueue` 引用队列一起使用,用于跟踪垃圾回收过程

在 `JDK1.2` 之后，用 `java.lang.ref.PhantomReference` 类来表示

```java
public class OOM {

    public static void main(String[] args) throws InterruptedException {
        testPhantomReference();
    }

    private static void testPhantomReference() throws InterruptedException {
        // 创建虚引用，并传入引用队列
        ReferenceQueue<PhantomReferenceDemo> queue = new ReferenceQueue<>();
        PhantomReference<PhantomReferenceDemo> ref = new PhantomReference<>(new PhantomReferenceDemo(),queue);

        // 虚引用的get始终为null
        System.out.println(ref.get());

        // 第一次gc ，观察日志得出JVM找到了垃圾对象，并调用finalize，但是并没有立即加入回收队列
        // Object [OOM$PhantomReferenceDemo@7ad74083] finalize...
        // null
        System.gc();
        System.out.println(queue.remove(200L));


        // 第二次gc , 观察日志得出JVM才真正清除垃圾对象，并将其加入引用队列（所以才能从队列中弹出值）
        // java.lang.ref.PhantomReference@2b193f2d
        System.gc();
        System.out.println(queue.remove(200L));


        // 第三次gc ， 与该对象已经无关
        // null
        System.gc();
        System.out.println(queue.remove(200L));

    }

    static class PhantomReferenceDemo {
        @Override
        protected void finalize() throws Throwable {
            super.finalize();
            System.out.println("Object ["+this+"] finalize...");
        }
    }
}
```

## 引用队列(ReferenceQueue)

引用队列可以与`软引用`、`弱引用`以及`虚引用`一起配合使用

当垃圾回收器准备回收一个对象时，如果发现它还有引用，那么就会在回收对象之前，把这个引用加入到与之关联的引用队列中去。程序可以通过判断引用队列中是否已经加入了引用，来判断被引用的对象是否将要被垃圾回收，这样就可以在对象被回收之前采取一些必要的措施.

与软引用、弱引用不同，`虚引用必须和引用队列一起使用`。

