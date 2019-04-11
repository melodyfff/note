---
title: 【Java】 hashcode()和System.identityHashCode()
tags: [Java]
date: 2019-4-11
---

# hashcode()和System.identityHashCode()

> openjdk8: http://hg.openjdk.java.net/jdk8u/jdk8u/jdk/file/5b86f66575b7

最近在看`Spring`源码的过程中看到这么一行

**@{link org.springframework.context.support.AbstractApplicationContext}**

```java
public AbstractApplicationContext() {
        this.logger = LogFactory.getLog(this.getClass());
        this.id = ObjectUtils.identityToString(this);
        this.displayName = ObjectUtils.identityToString(this);
        this.beanFactoryPostProcessors = new ArrayList();
        this.active = new AtomicBoolean();
        this.closed = new AtomicBoolean();
        this.startupShutdownMonitor = new Object();
        this.applicationListeners = new LinkedHashSet();
        this.resourcePatternResolver = this.getResourcePatternResolver();
}
```

在初始化`Context`时设置 `id` 和 `displayName`名字的时候 `ObjectUtils.identityToString(this)`

```java
    public static String identityToString(Object obj) {
        return obj == null ? "" : obj.getClass().getName() + "@" + getIdentityHexString(obj);
    }

    public static String getIdentityHexString(Object obj) {
        return Integer.toHexString(System.identityHashCode(obj));
    }
```

可以看到`Spring`的做法是：`类名 + @ + 16进制的字符串`


所以`System.identityHashCode()`是什么？

## hashcode()和System.identityHashCode()对比

来看个实例

```java
public class OK {
    public static void main(String[] args) {
        OK ok1 = new OK();
        OK ok2 = new OK();

        System.out.println("ok1 - hashCode : " + ok1.hashCode());// ok1 - hashCode : 1554874502
        System.out.println("ok2 - hashCode : " + ok2.hashCode());// ok2 - hashCode : 1846274136


        System.out.println("ok1 - System.identityHashCode : " + System.identityHashCode(ok1)); //ok1 - System.identityHashCode : 1554874502
        System.out.println("ok2 - System.identityHashCode : " + System.identityHashCode(ok2));//ok2 - System.identityHashCode : 1846274136
    }
}
```

从结果上来看，**相同对象的hashCode()和System.identityHashCode()是一致的**

接下来，我们覆盖下hashCode()

```java
public class OK {

    @Override
    public int hashCode() {
        return 1;
    }

    public int getSuperHashCode(){
        return super.hashCode();
    }

    public static void main(String[] args) {
        OK ok1 = new OK();
        OK ok2 = new OK();

        System.out.println("ok1 - hashCode : " + ok1.hashCode()); // ok1 - hashCode : 1
        System.out.println("ok2 - hashCode : " + ok2.hashCode()); // ok2 - hashCode : 1


        System.out.println("ok1 - System.identityHashCode : " + System.identityHashCode(ok1));//ok1 - System.identityHashCode : 1554874502
        System.out.println("ok2 - System.identityHashCode : " + System.identityHashCode(ok2));//ok2 - System.identityHashCode : 1846274136

        System.out.println("ok1 - SuperHashCode : " + ok1.getSuperHashCode());//ok1 - SuperHashCode : 1554874502
        System.out.println("ok2 - SuperHashCode : " + ok2.getSuperHashCode());//ok2 - SuperHashCode : 1846274136


    }
}
```

可以看到，如果重载了`hashCode()`方法，而又想获未重载之前的`object.hashCode()`,则可以使用`System.identityHashCode()`

## 深入System.identityHashCode()

openJDK8： http://hg.openjdk.java.net/jdk8u/jdk8u/jdk/file/5b86f66575b7

关于`System.identityHashCode()`里面的声明是这样的
```java
 /**
     * Returns the same hash code for the given object as
     * would be returned by the default method hashCode(),
     * whether or not the given object's class overrides
     * hashCode().
     * The hash code for the null reference is zero.
     *
     * @param x object for which the hashCode is to be calculated
     * @return  the hashCode
     * @since   JDK1.1
     */
    public static native int identityHashCode(Object x);
```

对于源码中的解读可以参考 [hashCode和identityHashCode底层是怎么生成的](https://www.cnblogs.com/godtrue/p/6395098.html)