---
title: 【Java】Spring中统一相同版本的api请求路径
tags: [java]
date: 2019-6-23
---

# Spring中统一相同版本的api请求路径

## 问题场景
当我们在实际开发中，可能会遇到开发相同同版本的`api`，

假设相同版本的`api`请求路径为`/v1/functionA`,`/v1/functionB`，

因为按照业务进行了划分,这两个对外暴露的`api`分别在两个不同的`Controller`中

因此可能存在以下代码

```java
@Controller
@RequestMapping("/v1")
public class FunctionControllerA{
    @RequestMapping("/functionA")
    public void functionA(){
        //Some thing to do...
    }
}

@Controller
@RequestMapping("/v1")
public class FunctionControllerB{
    @RequestMapping("/functionB")
    public void functionB(){
         //Some thing to do...
    }
}
```

可以看到我们在类上声明了`@RequestMapping("/v1")`这个注解，当然也可以在方法上声明如`@RequestMapping("/v1/functionA")`

如果这样的`Controller`不多可能还好，但如果存在很多个这样的情况，或者突然有一天上面突然让更换`/v1/...`开头的`api`地址改为`/v1.0/..`这样，是不是得更改很多个地方。

那么有没有版本统一一个地方，达到修改一次即可

## 解决方案

**声明一个统一api请求路径接口**

```java
@RequestMapping("/v1")
public interface VersionPathAware{}
```

上述两个地址可改写为以下形式

```java
@Controller
public class FunctionControllerA implements VersionPathAware {
    @RequestMapping("/functionA")
    public void functionA(){
        //Some thing to do...
    }
}

@Controller
public class FunctionControllerB implements VersionPathAware {
    @RequestMapping("/functionB")
    public void functionB(){
        //Some thing to do...
    }
}
```


`api`请求路径为`/v1/functionA`,`/v1/functionB`，

并且如果版本名需要更换只需更改一个地方即可。

**当然这种方法是一种约定,实际开发中也可直接使用第一种方式**