---
title: 【Java】 通过反射绕过泛型检查
tags: [Java]
date: 2019-4-23
---

# 通过反射绕过泛型检查

泛型用在编译期，编译过后泛型擦除（消失掉）。所以是可以通过反射越过泛型检查的

例如，有一个`String`的集合，怎样向这个集合内添加一个`Integer`的值？

```java
   public static void reflection() throws InvocationTargetException, IllegalAccessException, NoSuchMethodException {
        
        // 1. 新建存放String类型的ArrayList对象
        List<String> list = new ArrayList<>();
        list.add("hello");
        
        // 2. list直接添加123会报编译错误
        // list.add(123);
        
        // 3. 通过反射获取ArrayList的add方法，传参类型为Object.class
        Method add = list.getClass().getMethod("add", Object.class);
        
        // 4. 调用该方法传入123
        add.invoke(list, 123);
    }
```