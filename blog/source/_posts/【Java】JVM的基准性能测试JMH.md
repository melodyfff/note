---
title: 【Java】JVM的基准性能测试JMH
tags: [java]
date: 2020-7-29
---

# JVM的基准性能测试JMH
> JMH is a Java harness for building, running, and analysing nano/micro/milli/macro benchmarks written in Java and other languages targetting the JVM.

JMH 是一个由 OpenJDK/Oracle 里面那群开发了 Java 编译器的大牛们所开发的 `Micro Benchmark Framework` 。

何谓 `Micro Benchmark` 呢？简单地说就是在 `method` 层面上的 `benchmark（基准）`，精度可以精确到微秒级。

OpenJDK - JMH 官网文档：http://openjdk.java.net/projects/code-tools/jmh/

OpenJDK - JMH 官网example： http://hg.openjdk.java.net/code-tools/jmh/file/tip/jmh-samples/

## 快速创建一个JMH项目

创建项目
```bash
$ mvn archetype:generate \
          -DinteractiveMode=false \
          -DarchetypeGroupId=org.openjdk.jmh \
          -DarchetypeArtifactId=jmh-java-benchmark-archetype \
          -DgroupId=org.sample \
          -DartifactId=test \
          -Dversion=1.0
```
构建项目
```bash
$ cd test/
$ mvn clean install
```

运行项目
```bash
$ java -jar target/benchmarks.jar
```

## 参考博客

[Java 并发编程笔记：JMH 性能测试框架](http://blog.dyngr.com/blog/2016/10/29/introduction-of-jmh/)

[JMH: 最装逼，最牛逼的基准测试工具套件](https://www.jianshu.com/p/0da2988b9846)