---
title: 【Java】清除Java中的重复代码-骨架实现 
tags: [java]
date: 2019-3-24
---

# 清除Java中的重复代码-骨架实现 

> `Reference:`   
[在 Java 中应用骨架实现](https://mp.weixin.qq.com/s/7vNwfT_Ag6ftogSzumePDg)  
[Effective Java - ITEM 18 重组合，轻继承](https://github.com/XYliang/effective-java3_Chinese-version/issues/4)  
[Effective Java 3 相关博客](https://www.cnblogs.com/WutingjiaWill/p/9188809.html)

Java Skeletal Implementation/Abstract Interfaces(骨架实现/抽象接口)指通过接口和抽象类，集接口多继承的优势与抽象类可以减少重复代码的优势于一体。

`Java Collection API` 已经采用了这种设计：`AbstractSet`、 `AbstractMap` 等都是骨架实现案例。

`Joshua Bloch`的`Effective Java`书中也提到了骨架接口。

## 接口和抽象类

- **接口：** 可以实现多继承(`extends`)，但是不能有具体实现

- **抽象类：** 可以有具体实现，但是不能多继承(`extends`),可以实现(`implement`)多个接口


## 场景

对于一个咖啡续命的人来说，每天泡咖啡是常态，假如将我们泡咖啡的行为抽象则主要为以下步骤:

- 选择咖啡
- 泡咖啡
- 喝咖啡

### 方式一(接口实现)

**CoffeeDaily**

```java
public interface CoffeeDaily {
    void chooseCoffee();

    void makeCoffee();

    void drinkCoffee();

    void process();
}
```

**InstantCoffee**
```java
public class InstantCoffee implements CoffeeDaily{
    @Override
    public void chooseCoffee() {
        System.out.println("today is instant coffee.");
    }

    @Override
    public void makeCoffee() {
        System.out.println("make instant coffee");
    }

    @Override
    public void drinkCoffee() {
        System.out.println("drink instant coffee");
    }

    @Override
    public void process() {
        chooseCoffee();
        makeCoffee();
        drinkCoffee();
    }
}
```

**LatteCoffee**
```java
public class LatteCoffee implements CoffeeDaily{
    @Override
    public void chooseCoffee() {
        System.out.println("today is latte coffee.");
    }

    @Override
    public void makeCoffee() {
        System.out.println("make latte coffee");
    }

    @Override
    public void drinkCoffee() {
        System.out.println("drink latte coffee");
    }

    @Override
    public void process() {
        chooseCoffee();
        makeCoffee();
        drinkCoffee();
    }
}
```

**Daily**
```java
public class Daily {
    public static void main(String[] args) {
        InstantCoffee instantCoffee = new InstantCoffee();
        LatteCoffee latteCoffee = new LatteCoffee();

        instantCoffee.process();
        System.out.println("-----------------");
        latteCoffee.process();
    }
}
```

**output**
```bash
today is instant coffee.
make instant coffee
drink instant coffee
-----------------
today is latte coffee.
make latte coffee
drink latte coffee
```

由于实现接口的类都必须实现接口中定义的方法，所以可能会有重复的方法。

上述例子将所有步骤集中到`process()`中处理，但是其中其实有很多重复代码，单我们继续添加具体实现时，重复代码也会继续增加。

如果我们将公共的重复代码提取放入一个工具类中，看上去解决了大段重复代码的问题  

但是可能会造成`Shotgun surgery 问题代码`,而且也违反了`单一责任原则`

> [Shotgun surgery](https://en.wikipedia.org/wiki/Shotgun_surgery) 是软件开发中的一种反模式，它发生在开发人员向应用程序代码库添加特性的地方，这些代码库会在一次更改中跨越多个实现。

## 方式二(抽象类)

**AbstractCoffeeDaily**
```java
public abstract class AbstractCoffeeDaily {

    private String coffeeName;

    public AbstractCoffeeDaily(String coffeeName) {
        this.coffeeName = coffeeName;
    }

    public void chooseCoffee(){
        System.out.println((String.format("today is %s coffee.",coffeeName)));
    }

    public void makeCoffee(){
        System.out.println((String.format("make %s coffee",coffeeName)));
    }

    public void drinkCoffee(){
        System.out.println((String.format("drink %s coffee",coffeeName)));
    }

    protected final void process(){
        this.chooseCoffee();
        this.makeCoffee();
        this.drinkCoffee();
    }

}
```

**InstantCoffee**
```java
public class InstantCoffee extends AbstractCoffeeDaily{

    private static final String COFFEE_NAME = "instant";

    public InstantCoffee() {
        super(COFFEE_NAME);
    }
}
```

**LatteCoffee**
```java
public class LatteCoffee extends AbstractCoffeeDaily{

    private static final String COFFEE_NAME = "latte";

    public LatteCoffee() {
        super(COFFEE_NAME);
    }
}
```

**Daily**
```java
public class Daily {
    public static void main(String[] args) {
        AbstractCoffeeDaily instantCoffee = new InstantCoffee();
        AbstractCoffeeDaily latteCoffee = new LatteCoffee();

        instantCoffee.process();
        System.out.println("-----------------");
        latteCoffee.process();
    }
}
```

**output**
```bash
today is instant coffee.
make instant coffee
drink instant coffee
-----------------
today is latte coffee.
make latte coffee
drink latte coffee
```

由于菱形继承问题，Java 不支持多重继承。

**菱形继承问题：** 两个子类继承同一个父类，而又有子类又分别继承这两个子类，产生二义性问题

`InstantCoffee`和`LatteCoffee`都继承`AbstractCoffeeDaily`,抽取了公用代码，消除了重复的代码。

但是由于`Java`不支持多重继承，如果我此时想在添加一个外部类引入的方法，如：自动清洗杯子机器类`CupCleanMachine`。

此时的`InstantCoffee`和`LatteCoffee`不能再继续继承，如果采用组合(`composition`)的方式，将`CupCleanMachine`引入，则会造成强耦合。

## 方式三(骨架实现或抽象接口)

骨架实现的步骤：

- 创建接口
- 创建抽象类实现该接口，并实现公共方法
- 在具体实现子类中创建一个私有内部类，继承抽象类。将外部调用`委托`给抽象接口，该类可以在使用通用方法的同时继承和实现其他接口。

**接口：CoffeeDaily**
```java
public interface CoffeeDaily {
    void chooseCoffee();

    void makeCoffee();

    void drinkCoffee();

    void process();
}
```

**杯子清洗机器:CupCleanMachine**
```java
public class CupCleanMachine {
    public void clean(){
        System.out.println("clean the cup.");
    }
}
```

**AbstractCoffeeDaily**
```java
public class AbstractCoffeeDaily implements CoffeeDaily{

    private String coffeeName;

    public AbstractCoffeeDaily(String coffeeName) {
        this.coffeeName = coffeeName;
    }

    @Override
    public void chooseCoffee() {
        System.out.println((String.format("today is %s coffee.",coffeeName)));
    }

    @Override
    public void makeCoffee() {
        System.out.println((String.format("make %s coffee",coffeeName)));
    }

    @Override
    public void drinkCoffee() {
        System.out.println((String.format("drink %s coffee",coffeeName)));
    }

    @Override
    public final void process() {
        chooseCoffee();
        makeCoffee();
        drinkCoffee();
    }
}
```

**InstantCoffee**
```java
public class InstantCoffee implements CoffeeDaily{
    private static final String COFFEE_NAME = "instant";

    InstantCoffeeDelegator instantCoffeeDelegator = new InstantCoffeeDelegator(COFFEE_NAME);

    @Override
    public void chooseCoffee() {
        instantCoffeeDelegator.chooseCoffee();
    }

    @Override
    public void makeCoffee() {
        instantCoffeeDelegator.makeCoffee();
    }

    @Override
    public void drinkCoffee() {
        instantCoffeeDelegator.drinkCoffee();
    }

    @Override
    public void process() {
        instantCoffeeDelegator.process();
    }

    /** delegator AbstractCoffeeDaily general function */
    private class InstantCoffeeDelegator extends AbstractCoffeeDaily {
        public InstantCoffeeDelegator(String coffeeName) {
            super(coffeeName);
        }

    }
}
```

**LatteCoffee**
```java
public class LatteCoffee extends CupCleanMachine implements CoffeeDaily{

    private static final String COFFEE_NAME = "latte";

    LatteCoffeeDelegator latteCoffeeDelegator = new LatteCoffeeDelegator(COFFEE_NAME);

    @Override
    public void chooseCoffee() {
        latteCoffeeDelegator.chooseCoffee();
    }

    @Override
    public void makeCoffee() {
        latteCoffeeDelegator.makeCoffee();
    }

    @Override
    public void drinkCoffee() {
        latteCoffeeDelegator.drinkCoffee();
    }

    @Override
    public void process() {
        latteCoffeeDelegator.process();
        clean();
    }

    /** delegator AbstractCoffeeDaily general function */
    private class LatteCoffeeDelegator extends AbstractCoffeeDaily {
        public LatteCoffeeDelegator(String coffeeName) {
            super(coffeeName);
        }
    }
}
```

**Daily**
```java
public class Daily {
    public static void main(String[] args) {
        InstantCoffee instantCoffee = new InstantCoffee();
        LatteCoffee latteCoffee = new LatteCoffee();

        instantCoffee.process();
        System.out.println("-----------------");
        latteCoffee.process();
    }
}
```

**output**
```java
today is instant coffee.
make instant coffee
drink instant coffee
-----------------
today is latte coffee.
make latte coffee
drink latte coffee
clean the cup.
```

上述方法中先创建了一个接口，定义主要的行为，然后创建抽象类实现接口定义所有通用的方法实现。

在具体子类中，实现一个委托类(`delegator`),通过`delegator`调用通用的方法实现

这样子类即可以再继承其他类，又可以实现其他的接口，同时也消除了重复的通用方法。