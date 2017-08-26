---
title: 【JVM】偏向锁浅谈
tags: [java]
date: 2016-11-3
---
偏向锁是JDK1.6提出来的一种锁优化的机制。其核心的思想是，如果程序没有竞争，则取消之前已经取得锁的线程同步操作。也就是说，若某一锁被线程获取后，便进入偏向模式，当线程再次请求这个锁时，就无需再进行相关的同步操作了，从而节约了操作时间，如果在此之间有其他的线程进行了锁请求，则锁退出偏向模式。

## 偏向锁

在`JVM`中使用-XX:+UseBiasedLocking

```java
package jvm;

import java.util.List;
import java.util.Vector;
/**
 * Description：偏向锁
 * Created by ChenXin on 2016/11/3.
 */
public class Biased {
    public static List<Integer> numberList = new Vector<Integer>();
    public static void main(String[] args) {
        long begin = System.currentTimeMillis();
        int count = 0;
        int startnum = 0;
        while(count<10000000){
            numberList.add(startnum);
            startnum+=2;
            count++;
        }
        long end = System.currentTimeMillis();
        System.out.println("time:"+(end-begin));
    }
}
```
这里使用Vector而没用ArrayList,原因是ArrayList是线程不安全的，Vector是线程安全的。  
查看Vector的源码  
```java
/**
     * Returns the element at the specified position in this Vector.
     *
     * @param index index of the element to return
     * @return object at the specified index
     * @throws ArrayIndexOutOfBoundsException if the index is out of range
     *            ({@code index < 0 || index >= size()})
     * @since 1.2
     */
    public synchronized E get(int index) {
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        return elementData(index);
    }
```
查看ArrayList的源码  
```java
/**
 * Returns the element at the specified position in this list.
 *
 * @param  index index of the element to return
 * @return the element at the specified position in this list
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E get(int index) {
    rangeCheck(index);

    return elementData(index);
}
```
Vector中的几乎所有操作是带有sychronized的，而ArrayList是没有的，所以Vector是线程安全的。  
## 开启偏向锁和不开启偏向锁对程序性能的影响有多大
在没有修改`JVM`的参数的情况下：
> time:1675  

修改`-client -Xmx512m -Xms512m`
> time:497

使用偏向锁`-XX:+UseBiasedLocking -XX:BiasedLockingStartupDelay=0 -client -Xmx512m -Xms512m`
> time:357

开启偏向锁的程序运行时间明显较短，开启偏向锁比不开启偏向锁，在单个线程中操作一个对象的同步方法，是有一定的优势的。其实也可以这样理解，当只有一个线程操作带有同步方法的Vector对象的时候，此时对Vector的操作就转变成了对ArrayList的操作。  
偏向锁在锁竞争激烈的场合没有太强的优化效果，因为大量的竞争会导致持有锁的线程不停地切换，锁也很难保持在偏向模式，此时，使用偏向锁不仅得不到性能的优化，反而有可能降低系统的性能，因此，在激烈竞争的场合，可以尝试使用
`-XX:-UseBiastedLocking`参数禁用偏向锁。