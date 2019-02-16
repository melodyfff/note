---
title: 【Design Patterns】代理模式
tags: [Design Patterns]
date: 2019-2-12
---

# 代理模式

> 定义: 代理是一个包装器或代理对象，客户端正在调用它来访问幕后的真实服务对象。使用代理可以简单地转发到真实对象，或者可以提供额外的逻辑。

> wiki:https://en.wikipedia.org/wiki/Proxy_pattern

> 类型: 结构型模式

![](../img/W3sDesign_Proxy_Design_Pattern_UML.jpg)

> UML类图
```plantuml
@startuml proxy

class Client{

}

interface Subject{
    + method()
}

class Proxy{
    + method()
}

class RealSubject{
    + method()
}

Client ..> Subject

Subject <|-- RealSubject

Subject <|-- Proxy

Proxy ..> RealSubject

@enduml
```
![](../img/proxy_uml.png)


## 代理模式实例

### 静态代理

**结构如下:**

![](../img/static_proxy.png)

**抽象主题角色(公共接口)**
```java
public interface Subject {
    /** 具体要做的事 */
    void method();
}
```

**具体主题角色(真正的处理对象)**

```java
public class RealSubject implements Subject{
    /** 具体要做的事 */
    @Override
    public void method() {
        System.out.println("do something...");
    }
}
```

**代理主题角色(代理类)**
```java
public class Proxy implements Subject{
    /** 要代理的对象 */
    private Subject subject;

    /** 初始化时传入需要被代理的对象 */
    public Proxy(Subject subject){
        this.subject = subject;
    }

    /** 具体做的事情,可以拓展为算法骨架(模板方法) */
    @Override
    public void method() {
        this.before();
        this.subject.method();
        this.after();
    }

    /** 代理之前做的事情 */
    public void before(){
        System.out.println(">>> Proxy Before...");
    }

    /** 代理结束做的事情 */
    public void after(){
        System.out.println(">>> Proxy After...");
    }
}
```

**客户端(业务类)**
```java
    public static void main(String[] args) {
        RealSubject realSubject = new RealSubject();
        Proxy proxy = new Proxy(realSubject);
        proxy.method();
        // >>> Proxy Before...
        // do something...
        // >>> Proxy After...        
    }
```

## 动态代理

例如面向横切面编程-`AOP(Aspect Oriented Programming)`中就运用了动态代理机制.

### JDK实现

**UML**

```plantuml
@startuml
class Client{

}

class DynamicProxy{

}

interface InvocationHandler {
    + Object invoke()
}

class CustomInvocationHandler{
    + Object invoke()
}

interface Advice{
    + void exec()
}

class BeforeAdvice{
    + void exec()    
}


interface Subject{
    + void method()
}

class RealSubject{
    + void method()
}

Subject <|.. RealSubject

Client --> DynamicProxy

Client --> Subject

DynamicProxy --> InvocationHandler

DynamicProxy --> Advice 

Advice <|.. BeforeAdvice

InvocationHandler <|.. CustomInvocationHandler


note "Do Something before invoke" as N1
N1 .. Advice
@enduml
```

![](../img/dynamic_proxy.png)

#### 真正执行的主题

```java
public interface Subject {
    void method();
}

public class RealSubject implements Subject{

    @Override
    public void method() {
        System.out.println("do something...");
    }
}
```

#### 消息通知

```java
public interface Advice {
    void exec();
}

public class BeforeAdvice implements Advice {
    @Override
    public void exec() {
        System.out.println(">>> BeforeAdvice working...");
    }
}
```

#### 实现JDK的InvocationHandler
```java
public class CustomInvocationHandler implements InvocationHandler{

    // 被代理的对象
    private Object target;

    public CustomInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // proxy为生成的代理类
        // 调用被代理的对象的具体方法，以及传入参数
        return method.invoke(this.target,args);
    }
}
```

#### 动态代理类

```java
public final class DynamicProxy {

    /** 前置通知 */
    private static final Advice beforeAdvice = new BeforeAdvice();

    public static <T extends Subject> T newProxyInstance(Subject target){

        beforeAdvice.exec();
        
        // 传入 被代理类的类加载器ClassLoader | 被代理对象的实现接口 | InvocationHandler
        
        //noinspection unchecked
        return (T) Proxy.newProxyInstance(target.getClass().getClassLoader(),target.getClass().getInterfaces(),new CustomInvocationHandler(target));
        // 也可用lombda表达式  
        //return (T) Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), (proxy, method, args) -> method.invoke(target, args));
    }
}
```

#### 场景类
```java
public class Client {
    public static void main(String[] args) {
        final Subject o = DynamicProxy.newProxyInstance(new RealSubject());
        o.method();
        // >>> BeforeAdvice working...
        // do something...
    }
}
```

### CGLIB实现

****UML****

```plantuml
@startuml cglib proxy
class Subject{
   + method()
}

package net.sf.cglib <<Cloud>> {
interface MethodInterceptor{
  + intercept(Object var1, Method var2, Object[] var3, MethodProxy var4) throws Throwable:Object
}
}

class CglibProxyFactory{
  - target:Object
  + CglibProxyFactory(Object target)
  + getInstance():Object
  + intercept(Object var1, Method var2, Object[] var3, MethodProxy var4) throws Throwable:Object
}

class Client{
}

MethodInterceptor <..- CglibProxyFactory

Client --> CglibProxyFactory

Client --> Subject
@enduml
```

![](../img/cglib-proxy.png)

#### 主题类
```java
public class Subject {
    public String method(){
        System.out.println("Hello World!");
        return "ok";
    }
}
```

#### 代理工厂类
```java
// 1.继承接口MethodInterceptor ，override intercept()
public class CglibProxyFactory implements MethodInterceptor {

    /** 被代理对象 */
    private Object target;

    public CglibProxyFactory(Object target) {
        this.target = target;
    }

    public Object getInstance(){
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(this.target.getClass());
        enhancer.setCallback(this);

        // 创建代理对象
        return enhancer.create();
    }

    /**
     *
     * @param proxy cglib动态生成的代理实例
     * @param method 被代理的方法
     * @param args method方法调用传入参数
     * @param methodProxy 生成的代理类对方法的代理引用
     * @return 方法执行完成后的返回值
     * @throws Throwable
     */
    @Override
    public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {

        System.out.println("Transaction Open...");
        final Object invoke = method.invoke(target, args);
        System.out.println("Transaction Commit...");
        // 获取方法执行后的返回
        return invoke;
    }
}

// 2.采用匿名函数的方式实现
public class CglibProxyFactory2 {

    private Object target;

    public CglibProxyFactory2(Object target) {
        this.target = target;
    }

    public Object getInstance(){
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(target.getClass());
        enhancer.setCallback(new MethodInterceptor() {
            @Override
            public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
                System.out.println("Transaction Open...");
                final Object invoke = method.invoke(target, args);
                System.out.println("Transaction Commit...");
                // 获取方法执行后的返回
                return invoke;
            }
        });
        return enhancer.create();
    }
}
```

#### 场景类

```java
public class Client {
    public static void main(String[] args) {
        // ①. 真实对象
        Subject subject = new Subject();

        // 代理类的生成以及调用
        CglibProxyFactory proxy = new CglibProxyFactory(subject);
        final Subject proxyObject = (Subject) proxy.getInstance();
        proxyObject.method();
        // Transaction Open...
        // Hello World!
        // Transaction Commit...

        // 查看生成的代理类名字
        System.out.println(proxyObject.getClass());
        // class Subject$$EnhancerByCGLIB$$c156e2ba


        // 第二种形式代理类生成以及调用(由于代理的是同一对象，所以生成的代理类都是一致的)
        CglibProxyFactory2 factory2 = new CglibProxyFactory2(subject);

        final Subject instance = (Subject)factory2.getInstance();
        System.out.println(instance.method());
        // Transaction Open...
        // Hello World!
        // Transaction Commit...
        // ok


        System.out.println(factory2.getInstance().getClass());
        // class Subject$$EnhancerByCGLIB$$c156e2ba

    }
}
```

## 几种代理的对比

****静态代理：**** 由程序员或特定工具自动生成的源代码，再对其编译，在程序运行之前，代理的类编译生成的`.class`文件就已经存在了

****动态代理:**** 在程序运行时，通过反射机制动态创建而成。

| 代理方式        | 具体实现   |  优点  | 缺点| 底层
| --------   | :----  | :----  | :----|:----|
| 静态代理     | 代理类与委托类实现同一接口<br>在代理类中需要硬编码接口 |   实现简单<br>容易理解    | 代理类需要硬编码接口<br>在实际应用中可能会导致重复编码<br>浪费存储空间并且效率很低|
| JDK动态代理     |   代理类与委托类实现同一接口<br>主要是通过代理类实现`InvocationHandler`并重写`invoke`方法来进行动态代理<br>在invoke方法中将对方法进行增强处理   |   不需要硬编码接口<br>代码复用率高   |只能够代理实现了接口的委托类|用反射机制进行方法的调用|
| CGLIB动态代理   |   代理类将委托类作为自己的父类并为其中的`非final`委托方法创建两个方法<br>一个是与委托方法签名相同的方法，它在方法中会通过super调用委托方法<br>另一个是代理类独有的方法<br>在代理方法中，它会判断是否存在实现了`MethodInterceptor`接口的对象，若存在则将调用`intercept`方法对委托方法进行代理|  可以在运行时对类或者是接口进行增强操作<br>委托类无需实现接口  |不能对`final类`以及`final方法`进行代理|底层将方法全部存入一个数组中,<br>通过数组索引直接进行方法调用|
