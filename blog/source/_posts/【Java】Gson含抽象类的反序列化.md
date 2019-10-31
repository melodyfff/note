---
title: 【Java】Gson含抽象类的反序列化
tags: [java]
date: 2019-10-31
---
# Gson含抽象类的反序列化

场景描述:

`反序列化A类的时候，这个类里面有一个抽象类属性B，B的实现类C里面又有一个抽象类属性D，D的实现类是E`
## 实体类准备
```java
public class A implements Serializable {
    private B b;

    public A(B b) {
        this.b = b;
    }

    @Override
    public String toString() {
        return "A{" +
                "b=" + b +
                '}';
    }
}
public abstract class B implements Serializable {
    protected String name;

    public B(){}
    public B(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "B{" +
                "name='" + name + '\'' +
                '}';
    }
}
public class C extends B{
    private D d;

    public C(String name) {
        super(name);
    }
    public C(String name,D d) {
        super(name);
        this.d = d;
    }

    @Override
    public String toString() {
        return "C{" +
                "d=" + d +
                '}';
    }
}
public abstract class D implements Serializable {
    protected String name;

    public D(){}
    public D(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "D{" +
                "name='" + name + '\'' +
                '}';
    }
}
public class E extends D{
    public E() {
        super("this is E");
    }

    public E(String name, String name1) {
        super(name);
        this.name = name1;
    }

    @Override
    public String toString() {
        return "E{" +
                "name='" + name + '\'' +
                '}';
    }
}
```

## 反序列化适配器
```java
public class Adapter<T> implements JsonDeserializer<T> {
    private T obj;

    @SuppressWarnings("unchecked")
    public Adapter(Class<?> clz) {
        this.obj = (T) clz;
    }

    @Override
    public T deserialize(JsonElement jsonElement, Type type, JsonDeserializationContext context) throws JsonParseException {
        try {
            return OK.GsonFactory.gson.fromJson(jsonElement, (Type) Class.forName(((Class) obj).getName()));
        } catch (ClassNotFoundException e) {
            throw new JsonParseException(e);
        }
    }
}
```
## 反序列化
```java
public class OK {
    public static void main(String[] args) {
        E e = new E("this is e","this e is filed");
        C c = new C("this is c", e);
        final A a = new A(c);

        final A a1 = GsonFactory.gson.fromJson(new Gson().toJson(a), A.class);

        System.out.println(a);  // A{b=C{d=E{name='this e is filed'}}}
        System.out.println(a1);  // A{b=C{d=E{name='this e is filed'}}}
    }

    public static class GsonFactory {
        public static Gson gson;
        static {
            gson = new GsonBuilder()
                    // 指定对应抽象类的具体处理类型
                    .registerTypeAdapter(B.class,new Adapter<C>(C.class))
                    .registerTypeAdapter(D.class,new Adapter<E>(E.class))
                    .create();
        }
    }
}
```