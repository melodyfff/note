---
title: 【Design Patterns】建造者模式
tags: [Design Patterns]
date: 2018-5-28
---



# 建造者模式

最近在对项目进行重构工作的时候，发现为了打印规范日志，不断的去`new Object()`、`set()`，因为日志规范原因。不同功能模块的记录的日志的某些属性是不一致的，于是就出现了如下类似的情况：

```java
LogObject object = new LogObject();
object.set...
object.set...
object.set...
object.set...
...
```

想到`apache`的`httpclient`中`HttpClients`对象中使用了建造者模式,刚好也时候当前场景。

> 关于建造者模式的定义如下:
> - 建造者模式也称为生成器模式，是一种对象创建型模式。
> - 它可以将复杂对象的建造过程抽象出来（抽象类别），使这个抽象过程的不同实现方法可以构造出不同表现（属性）的对象。

## 建造者模式的角色

#### `Builder(建造者角色)`

- 对复杂对象的创建过程加以抽象，通过一个抽象接口来规范各个组成部分的构建

#### `ConcreteBuilder(具体创建者角色)`

- 实现`Builder`接口，针对不同的业务逻辑，具体化复杂对象的各个部分的创建，构建结束后，返回产品对象

#### `Director(指导者)`

- 构建一个使用`Builder`接口的对象

#### `Product(产品)`

- 要创建的复杂对象

## 建造者模式实例

#### Product

```java
/**
 * Product(产品对象)
 */
public class LogObject {

    private String name;

    private String content;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }
}
```

#### Builder
```java
/**
 * Builder(建造者角色)
 */
public interface Builder {

    /**
     * 创建日志对象
     * @return LogObject
     */
    LogObject build();

    /**
     * 创建日志对象，并打印日志对象
     * @return LogObject
     */
    LogObject buildAndLog();
}
```

#### ConcreteBuilder
```java
/**
 * ConcreteBuilder(具体创建者角色)
 */
public class LogObjectBuilder implements Builder{

    private transient static final Logger log = LoggerFactory.getLogger(LogObjectBuilder.class.getName());

    private LogObject logObject;

    protected LogObjectBuilder() {
        this.logObject = new LogObject();
    }

    public static LogObjectBuilder create(){
        return new LogObjectBuilder();
    }

    @Override
    public LogObject build() {
        return logObject;
    }

    @Override
    public LogObject buildAndLog() {
        log.info(Objects.toString(logObject));
        return logObject;
    }


    public final LogObjectBuilder setName(String name) {
        this.logObject.setName(name);
        return this;
    }

    public final LogObjectBuilder setContent(String content) {
        this.logObject.setContent(content);
        return this;
    }
}
```

#### Director
```java
/**
 * Director(指导者)
 */
public class LogObjects {

    private LogObjects(){}

    public static LogObjectBuilder custom(){
        return LogObjectBuilder.create();
    }

    public static LogObject createDefault(){
        return LogObjectBuilder.create().build();
    }
}
```


#### 使用

```java
    public static void main(String[] args) {
        System.out.println(LogObjects.createDefault());

        final LogObject test = LogObjects.custom().setName("test").build();
        System.out.println(test.getName());

        LogObjects.custom()
                .setName("test2")
                .setContent("ok")
                .buildAndLog();
    }
```
