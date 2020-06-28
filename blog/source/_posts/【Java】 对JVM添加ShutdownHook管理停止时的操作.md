---
title: 【Java】 对JVM添加ShutdownHook管理停止时的操作
tags: [java]
date: 2020-6-28
---

# 对JVM添加ShutdownHook管理停止时的操作

当`JVM`准备停止时，添加`ShutdownHook`监听状态，可处理一些对象的生命周期

## 简单例子
```java
public class Demo {

    public static void main(String[] args) throws IOException {
        System.out.println("Start.");

        // adding hook
        Runtime.getRuntime().addShutdownHook(new Thread(new Hook(),"hook"));
        Runtime.getRuntime().addShutdownHook(new Thread(new Hook2(),"hook2"));

        // imitate do something...
        System.in.read();

        System.out.println("Shutdown.");
    }


    static class Hook implements Runnable{
        @Override
        public void run() {
            // 当jvm准备停止时执行此hook
            System.out.println(Thread.currentThread().getName()+": Hook Running.");
        }
    }

    static class Hook2 implements Runnable{
        @Override
        public void run() {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            // 当jvm准备停止时执行此hook
            System.out.println(Thread.currentThread().getName()+": Hook Running.");
        }
    }
}
```
