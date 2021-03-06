---
title: 【Java】 浅析Java对象的拷贝
tags: [java]
date: 2019-01-01
---
> 前言：  
> 在平常开发中，常常遇到这种情况： 存在对象A，里面包含了一些初始化值，此时需要一个和A完全相同的对象B，并且之后对B的操作和改动都不会影响A。A和B为相互独立的对象。

实现对象克隆有两种方式:

- ① 实现Cloneable接口并重写Object类中的clone()方法；

- ② 实现Serializable接口，通过对象的序列化和反序列化实现克隆，可以实现真正的深度克隆。

> 注意：基于序列化和反序列化实现的克隆不仅仅是深度克隆，更重要的是通过泛型限定，可以检查出要克隆的对象是否支持序列化，这项检查是编译器完成的，不是在运行时抛出异常，这种是方案明显优于使用Object类的clone方法克隆对象。让问题在编译的时候暴露出来总是优于把问题留到运行时。  
> 参考:  
> [Java提高篇——对象克隆（复制）](https://www.cnblogs.com/Qian123/p/5710533.html)  

## 原始数据类型
七种原始数据类型(`boolean,char,byte,short,float,double.long`)的复制比较简单，可以通过直接赋值进行。

如下：
```java
int a = 6; // a: 6
int b = a; // b: 6

b = 7; // b: 7 ,a: 6
```

## Object的Clone
#### Coffee类
```java
public class Coffee {
    private String coffeeName;

    public Coffee(String coffeeName) {
        this.coffeeName = coffeeName;
    }

    public String getCoffeeName() {
        return coffeeName;
    }

    public void setCoffeeName(String coffeeName) {
        this.coffeeName = coffeeName;
    }
}
```

此时我们尝试复制`Coffee`类，如下:
```java
public static void main(String[] args) {
    Coffee a = new Coffee("Latte");
    Coffee b = a;

    System.out.println(a); // Coffee@29453f44
    System.out.println(b); // Coffee@29453f44
}
```
发现直接赋值后 `a和b`指向内存堆中的同一对象，也就是说，`a或者b做出改动，都会对对方造成影响。`

那么该如何达到复制一个对象的目的呢?

创世神`Object`中有两个为`protectd`的方法，分别为`finalize()`和`clone()`
```java
/**
     * Creates and returns a copy of this object.  The precise meaning
     * of "copy" may depend on the class of the object. The general
     * intent is that, for any object {@code x}, the expression:
     * <blockquote>
     * <pre>
     * x.clone() != x</pre></blockquote>
     * will be true, and that the expression:
     * <blockquote>
     * <pre>
     * x.clone().getClass() == x.getClass()</pre></blockquote>
     * will be {@code true}, but these are not absolute requirements.
     * While it is typically the case that:
     * <blockquote>
     * <pre>
     * x.clone().equals(x)</pre></blockquote>
     * will be {@code true}, this is not an absolute requirement.
     * <p>
     * By convention, the returned object should be obtained by calling
     * {@code super.clone}.  If a class and all of its superclasses (except
     * {@code Object}) obey this convention, it will be the case that
     * {@code x.clone().getClass() == x.getClass()}.
     * <p>
     * By convention, the object returned by this method should be independent
     * of this object (which is being cloned).  To achieve this independence,
     * it may be necessary to modify one or more fields of the object returned
     * by {@code super.clone} before returning it.  Typically, this means
     * copying any mutable objects that comprise the internal "deep structure"
     * of the object being cloned and replacing the references to these
     * objects with references to the copies.  If a class contains only
     * primitive fields or references to immutable objects, then it is usually
     * the case that no fields in the object returned by {@code super.clone}
     * need to be modified.
     * <p>
     * The method {@code clone} for class {@code Object} performs a
     * specific cloning operation. First, if the class of this object does
     * not implement the interface {@code Cloneable}, then a
     * {@code CloneNotSupportedException} is thrown. Note that all arrays
     * are considered to implement the interface {@code Cloneable} and that
     * the return type of the {@code clone} method of an array type {@code T[]}
     * is {@code T[]} where T is any reference or primitive type.
     * Otherwise, this method creates a new instance of the class of this
     * object and initializes all its fields with exactly the contents of
     * the corresponding fields of this object, as if by assignment; the
     * contents of the fields are not themselves cloned. Thus, this method
     * performs a "shallow copy" of this object, not a "deep copy" operation.
     * <p>
     * The class {@code Object} does not itself implement the interface
     * {@code Cloneable}, so calling the {@code clone} method on an object
     * whose class is {@code Object} will result in throwing an
     * exception at run time.
     *
     * @return     a clone of this instance.
     * @throws  CloneNotSupportedException  if the object's class does not
     *               support the {@code Cloneable} interface. Subclasses
     *               that override the {@code clone} method can also
     *               throw this exception to indicate that an instance cannot
     *               be cloned.
     * @see java.lang.Cloneable
     */
    protected native Object clone() throws CloneNotSupportedException;
```

从`Java`源码中可以看出`clone()`为一个`native`方法。

`native`介绍：https://www.cnblogs.com/Qian123/p/5702574.html


`native 即 JNI,Java Native Interface,是Java程序运行在JVM上时，访问比较底层的与操作系统相关的代码(C/C++)的API`

在源码注释中可以看出:

- `1) x.clone() != x will be true` 
- `2) x.clone().getClass() == x.getClass() will be true, but these are not absolute requirements`
- `3) x.clone().equals(x) will be true, this is not an absolute requirement`

1) 克隆对象将有单独的内存地址分配
2) 原始和克隆的对象应该具有相同的类类型，但它不是强制性的
3) 原始和克隆的对象应该是平等的equals()方法使用，但它不是强制性的

因为每个类的父类直接或者间接都是`Object`,因此他们都含有`clone()`方法。但是因为该方法是`protected`，所以都不能在类外进行访问。

我们常见的`Object a=new Object();Object b;b=a;`这种形式的代码复制的是引用，即对象在内存中的地址，a和b对象仍然指向了同一个对象。

因此可以通过`clone()`方法复制一个和原来对象同时独立存在的对象。

### 浅克隆(ShallowClone)

- 1. 实现`Cloneable`接口，否则在调用`clone()`方法时会抛出`CloneNotSupportedException`
- 2. 覆盖`clone()`方法，修饰符可根据后续需要改为`public`或者依旧为`protected`
- 3. 调用`super.clone()`即可获取克隆对象。

#### Coffee类
```java
public class Coffee implements Cloneable{
    /** 咖啡名 */
    private String coffeeName;
    /** 产地 */
    private Origin origin;

    public Coffee(String coffeeName, String origin) {
        this.coffeeName = coffeeName;
        this.origin = new Origin(origin);
    }

    @Override
    public Coffee clone(){
        Coffee coffee = null;
        try {
            coffee =  (Coffee) super.clone();
        } catch (CloneNotSupportedException e){
            e.printStackTrace();
        }
        return coffee;
    }

    public String getCoffeeName() {
        return coffeeName;
    }

    public void setCoffeeName(String coffeeName) {
        this.coffeeName = coffeeName;
    }

    public Origin getOrigin() {
        return origin;
    }

    public void setOrigin(Origin origin) {
        this.origin = origin;
    }
}

class Origin{
    /** 产地 */
    private String originName;

    public Origin(String originName) {
        this.originName = originName;
    }

    public String getOriginName() {
        return originName;
    }

    public void setOriginName(String originName) {
        this.originName = originName;
    }
}
```

结果如下:
```java
    public static void main(String[] args){
        Coffee a = new Coffee("Latte","China");
        Coffee b = a.clone();

        System.out.println(a); // Coffee@1d44bcfa
        System.out.println(b); // Coffee@266474c2

        System.out.println(a.getOrigin()); // Origin@6f94fa3e
        System.out.println(b.getOrigin()); // Origin@6f94fa3e
    }
```

从上可以看出虽然`Coffee`类实现了克隆,但是对于`引用的对象`实际上还是指向的同一个。

### 深克隆(DeepClone)
#### Coffee类
```java
public class Coffee implements Cloneable{

    private String coffeeName;

    private Origin origin;

    public Coffee(String coffeeName, String origin) {
        this.coffeeName = coffeeName;
        this.origin = new Origin(origin);
    }

    /**
     * 覆盖父类clone()
     * @return Coffee
     */
    @Override
    public Coffee clone(){
        Coffee coffee = null;
        try {
            coffee =  (Coffee) super.clone();
            // 深度clone
            coffee.origin = this.origin.clone();
        } catch (CloneNotSupportedException e){
            e.printStackTrace();
        }
        return coffee;
    }

    public String getCoffeeName() {
        return coffeeName;
    }

    public void setCoffeeName(String coffeeName) {
        this.coffeeName = coffeeName;
    }

    public Origin getOrigin() {
        return origin;
    }

    public void setOrigin(Origin origin) {
        this.origin = origin;
    }
}

class Origin implements Cloneable{

    private String originName;

    public Origin(String originName) {
        this.originName = originName;
    }
    /**
     * 覆盖父类的clone()
     * @return Origin.clone()
     * @throws CloneNotSupportedException CloneNotSupportedException
     */
    @Override
    protected Origin clone() throws CloneNotSupportedException {
        return (Origin) super.clone();
    }

    public String getOriginName() {
        return originName;
    }

    public void setOriginName(String originName) {
        this.originName = originName;
    }
}
```

结果如下：
```java
    public static void main(String[] args){
        Coffee a = new Coffee("Latte","China");
        Coffee b = a.clone();

        System.out.println(a); // Coffee@1d44bcfa
        System.out.println(b); // Coffee@266474c2

        System.out.println(a.getOrigin()); // Origin@6f94fa3e
        System.out.println(b.getOrigin()); // Origin@5e481248
    }
```

### 序列化的方式实现克隆

> Java语言提供的Cloneable接口和Serializable接口的代码非常简单，它们都是空接口，这种空接口也称为标识接口，标识接口中没有任何方法的定义，其作用是告诉JRE这些接口的实现类是否具有某个功能，如是否支持克隆、是否支持序列化等

```java
public class Coffee implements Serializable{

    private static final long serialVersionUID = 1L;

    private String coffeeName;

    private Origin origin;

    public Coffee(String coffeeName, String origin) {
        this.coffeeName = coffeeName;
        this.origin = new Origin(origin);
    }
    /**
     * 自定义序列号clone
     * @return new Coffee clone
     */
    public Coffee serialClone(){
        Coffee coffee = null;
        try {
            // ① 将当前对象写入流
            ByteArrayOutputStream bot = new ByteArrayOutputStream();
            ObjectOutputStream oos = new ObjectOutputStream(bot);
            oos.writeObject(this);
            
            // ② 从流中读取对象
            ByteArrayInputStream bin = new ByteArrayInputStream(bot.toByteArray());
            ObjectInputStream ois = new ObjectInputStream(bin);
            coffee = (Coffee) ois.readObject();
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
        return coffee;
    }

    public String getCoffeeName() {
        return coffeeName;
    }

    public void setCoffeeName(String coffeeName) {
        this.coffeeName = coffeeName;
    }

    public Origin getOrigin() {
        return origin;
    }

    public void setOrigin(Origin origin) {
        this.origin = origin;
    }
}

class Origin implements Serializable{

    private static final long serialVersionUID = 1L;

    private String originName;

    public Origin(String originName) {
        this.originName = originName;
    }
    public String getOriginName() {
        return originName;
    }

    public void setOriginName(String originName) {
        this.originName = originName;
    }
}
```

结果如下：
```java
    public static void main(String[] args){
        Coffee a = new Coffee("Latte","China");
        Coffee b = a.serialClone();

        System.out.println(a);  // Coffee@4dc63996
        System.out.println(b);  // Coffee@27973e9b

        System.out.println(a.getOrigin());  // Origin@4b1210ee
        System.out.println(b.getOrigin());  // Origin@312b1dae
    }
```

当然也可以使用第三方工具序列化如`fastjson`、`jackson`