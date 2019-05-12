---
title: 【Java】深入理解Java中的spi机制
tags: [Java]
date: 2019-5-12
---

# 深入理解Java中的spi机制

`SPI`全名为`Service Provider Interface`是JDK内置的一种服务提供发现机制,是Java提供的一套用来被第三方实现或者扩展的API，它可以用来启用框架扩展和替换组件。

`JAVA SPI` = **基于接口的编程＋策略模式＋配置文件 的动态加载机制**


**Java SPI的具体约定如下：**

当服务的提供者，提供了服务接口的一种实现之后，在`jar`包的`META-INF/services/`目录里同时创建一个以服务接口命名的文件。该文件里就是实现该服务接口的具体实现类。

而当外部程序装配这个模块的时候，就能通过该`jar`包`META-INF/services/`里的配置文件找到具体的实现类名，并装载实例化，完成模块的注入。

根据SPI的规范我们的服务实现类必须有一个`无参构造方法`。

为什么一定要在`classes`中的`META-INF/services`下呢？

JDK提供服务实现查找的一个工具类：`java.util.ServiceLoader`

在这个类里面已经写死

```java
// 默认会去这里寻找相关信息
private static final String PREFIX = "META-INF/services/";
```


**常见的使用场景：**
- `JDBC`加载不同类型的数据库驱动
- 日志门面接口实现类加载，`SLF4J`加载不同提供商的日志实现类
- `Spring`中大量使用了`SPI`,
    - 对`servlet3.0`规范
    - 对`ServletContainerInitializer`的实现
    - 自动类型转换`Type Conversion SPI(Converter SPI、Formatter SPI)`等
- `Dubbo`里面有很多个组件，每个组件在框架中都是以接口的形成抽象出来！具体的实现又分很多种，在程序执行时根据用户的配置来按需取接口的实现

## 简单的spi实例

整体包结构如下
```bash
└─main
    ├─java
    │  └─com
    │      └─xinchen
    │          └─spi
    │              └─App.java
    │              └─IService.java
    │              └─ServiceImplA.java
    │              └─ServiceImplB.java            
    └─resources
        └─META-INF
            └─services
                └─com.xinchen.spi.IService
```

**SPI接口**
```java
public interface IService {
    void say(String word);
}
```
**具体实现类**
```java
public class ServiceImplA implements IService {
    @Override
    public void say(String word) {
        System.out.println(this.getClass().toString() + " say: " + word);
    }
}

public class ServiceImplB implements IService {
    @Override
    public void say(String word) {
        System.out.println(this.getClass().toString() + " say: " + word);
    }
}
```

**/resource/META-INF/services/com.xinchen.spi.IService**
```bash
com.xinchen.spi.ServiceImplA
com.xinchen.spi.ServiceImplB
```

**Client类**
```java
public class App {
    static ServiceLoader<IService> services = ServiceLoader.load(IService.class);

    public static void main(String[] args) {
        for (IService service:services){
            service.say("Hello World!");
        }
    }
}

//    结果：
//    class com.xinchen.spi.ServiceImplA say: Hello World!
//    class com.xinchen.spi.ServiceImplB say: Hello World!
```

## 源码解析

`java.util.ServiceLoader`中的Fied区域
```java
    // 加载具体实现类信息的前缀
    private static final String PREFIX = "META-INF/services/";

    // 需要加载的接口
    // The class or interface representing the service being loaded
    private final Class<S> service;

    // 用于加载的类加载器
    // The class loader used to locate, load, and instantiate providers
    private final ClassLoader loader;

    // 创建ServiceLoader时采用的访问控制上下文
    // The access control context taken when the ServiceLoader is created
    private final AccessControlContext acc;

    // 用于缓存已经加载的接口实现类，其中key为实现类的完整类名
    // Cached providers, in instantiation order
    private LinkedHashMap<String,S> providers = new LinkedHashMap<>();

    // 用于延迟加载接口的实现类
    // The current lazy-lookup iterator
    private LazyIterator lookupIterator;
```



从`ServiceLoader.load(IService.class)`进入源码中

```java
    public static <S> ServiceLoader<S> load(Class<S> service) {
        // 获取当前线程上下文的类加载器
        ClassLoader cl = Thread.currentThread().getContextClassLoader();
        return ServiceLoader.load(service, cl);
    }
```

在`ServiceLoader.load(service, cl)`中
```java
    public static <S> ServiceLoader<S> load(Class<S> service,ClassLoader loader){
        // 返回ServiceLoader的实例
        return new ServiceLoader<>(service, loader);
    }

    private ServiceLoader(Class<S> svc, ClassLoader cl) {
        service = Objects.requireNonNull(svc, "Service interface cannot be null");
        loader = (cl == null) ? ClassLoader.getSystemClassLoader() : cl;
        acc = (System.getSecurityManager() != null) ? AccessController.getContext() : null;
        reload();
    }

    public void reload() {
        // 清空已经缓存的加载的接口实现类
        providers.clear();
        // 创建新的延迟加载迭代器
        lookupIterator = new LazyIterator(service, loader);
    }   
    
    private LazyIterator(Class<S> service, ClassLoader loader) {
        // 指定this类中的 需要加载的接口service和类加载器loader
        this.service = service;
        this.loader = loader;
    }         
```

当我们通过迭代器获取对象实例的时候，首先在成员变量`providers`中查找是否有缓存的实例对象

如果存在则直接返回，否则则调用`lookupIterator`延迟加载迭代器进行加载

迭代器判断的代码如下
```java
public Iterator<S> iterator() {
        // 返回迭代器
        return new Iterator<S>() {
            // 查询缓存中是否存在实例对象
            Iterator<Map.Entry<String,S>> knownProviders
                = providers.entrySet().iterator();

            public boolean hasNext() {
                // 如果缓存中已经存在返回true
                if (knownProviders.hasNext())
                    return true;
                // 如果不存在则使用延迟加载迭代器进行判断是否存在
                return lookupIterator.hasNext();
            }

            public S next() {
                // 如果缓存中存在则直接返回
                if (knownProviders.hasNext())
                    return knownProviders.next().getValue();
                // 调用延迟加载迭代器进行返回
                return lookupIterator.next();
            }

            public void remove() {
                throw new UnsupportedOperationException();
            }

        };
    }
```

**LazyIterator的类加载**
```java
        // 判断是否拥有下一个实例
        private boolean hasNextService() {
            // 如果拥有直接返回true
            if (nextName != null) {
                return true;
            }

            // 具体实现类的全名 ，Enumeration<URL> config
            if (configs == null) {
                try {
                    String fullName = PREFIX + service.getName();
                    if (loader == null)
                        configs = ClassLoader.getSystemResources(fullName);
                    else
                        configs = loader.getResources(fullName);
                } catch (IOException x) {
                    fail(service, "Error locating configuration files", x);
                }
            }
            while ((pending == null) || !pending.hasNext()) {
                if (!configs.hasMoreElements()) {
                    return false;
                }
                // 转换config中的元素，或者具体实现类的真实包结构
                pending = parse(service, configs.nextElement());
            }
            // 具体实现类的包结构名
            nextName = pending.next();
            return true;
        }

        private S nextService() {
            if (!hasNextService())
                throw new NoSuchElementException();
            String cn = nextName;
            nextName = null;
            Class<?> c = null;
            try {
                // 加载类对象
                c = Class.forName(cn, false, loader);
            } catch (ClassNotFoundException x) {
                fail(service,
                     "Provider " + cn + " not found");
            }
            if (!service.isAssignableFrom(c)) {
                fail(service,
                     "Provider " + cn  + " not a subtype");
            }
            try {
                // 通过c.newInstance()实例化
                S p = service.cast(c.newInstance());
                // 将实现类加入缓存
                providers.put(cn, p);
                return p;
            } catch (Throwable x) {
                fail(service,
                     "Provider " + cn + " could not be instantiated",
                     x);
            }
            throw new Error();          // This cannot happen
        }
```

## 总结

**优点**

使用Java SPI机制的优势是实现解耦，使得第三方服务模块的装配控制的逻辑与调用者的业务代码分离，而不是耦合在一起。应用程序可以根据实际业务情况启用框架扩展或替换框架组件。

**缺点**

- 多个并发多线程使用ServiceLoader类的实例是不安全的

- 虽然ServiceLoader也算是使用的延迟加载，但是基本只能通过遍历全部获取，也就是接口的实现类全部加载并实例化一遍。

## 参考

[The Java™ Tutorials](https://docs.oracle.com/javase/tutorial/ext/basics/spi.html)

[聊聊Dubbo（五）：核心源码-SPI扩展](https://www.jianshu.com/p/7daa38fc9711)

[深入理解Java SPI机制](https://shuaijunlan.github.io/2018/08/03/java-spi-introduction/)