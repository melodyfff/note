---
title: 【Design Patterns】单例模式
tags: [Design Patterns]
date: 2018-5-28
---

# 单例模式
什么是单例？表面意思就是一个实例，只希望有一个这样的实例存在。

## 懒加载（懒汉）

```java
public class Singleton {
    // volatile 防止编译优化打乱顺序,造成线程安全问题
    private volatile static Singleton instance;

    private Singleton(){}

    public static Singleton getInstance(){
        // 双重检查
        if (instance == null){
            synchronized (Singleton.class){
                if (instance == null){
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

静态内部类

```java
public class Singleton {
    private Singleton(){}

    // Java中静态内部类可以访问其外部类的成员属性和方法，同时，静态内部类只有当被调用的时候才开始首次被加载
    private static class SingletonHolder{
        public static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance(){
        return SingletonHolder.INSTANCE;
    }
}
```

## 饿汉

```java
public class Singleton {
    // 尚未使用时就初始化
    private static Singleton instance = new Singleton();

    private Singleton(){}

    public static Singleton getInstance(){
        return instance;
    }
}
```

## 枚举
`enum` 为 `final`,有且仅有`private`的构造器，自由序列化，线程安全，保证单例  
JVM可以保证枚举类型不可被反射


```java
public enum  Singleton {
    INSTANCE{
        @Override
        protected void answer() {
            System.out.println("yeah");
        }
    };

    public void ask(){
        System.out.println("hello world");
    }

    protected abstract void answer();
}
```

测试代码
```java
public class Run {
    public static void main(String[] args) {
        Singleton instance = Singleton.INSTANCE;
        Singleton instance1 = Singleton.INSTANCE;

        System.out.println(instance.hashCode());
        System.out.println(instance1.hashCode());
        System.out.println(instance.equals(instance1));
    }
}
```