---
title: 【Java】Java 日期类
tags: [java]
date: 2017-8-27
---

 > java.util.Date  //Sun Aug 27 21:56:53 CST 2017
 > java.sql.Date  //2017-08-27
 > java.sql.Time //22:21:02
 > java.sql.Timestamp //2017-08-27 22:18:23.857
 > java.text.SimpleDateFormat //格式化
 > java.util.Calendar  	//日历类
 
## java.util.Date是java.sqlDate,Time,Timestamp的父类
 
```java
java.util.Date date = new java.util.Date();
//Sun Aug 27 21:56:53 CST 2017
```
 
## java.sql.Date 是针对SQL语句使用的，new java.sql.Date(new java.util.Date().getTime()，它只包含日期而没有时间部分

```java
java.sql.Date date = new java.sql.Date(1111111111);//1970-01-14
java.sql.Date date2= new java.sql.Date(new java.util.Date().getTime());//2017-08-27
```

## 都有getTime方法返回毫秒数
```java
//1503843274139
java.sql.Date.getTime()
java.util.Date().getTime()
java.sql.Time.getTime()
java.sql.Timestamp.getTime()
```

## java.sql.Date和java.util.Date互相转换

```java
new java.sql.Date(new java.util.Date().getTime())
new java.util.Date(new java.sql.Date(0).getTime())
```

## java.sql.Timestamp的使用
```java
java.util.Date date = new java.util.Date();
java.sql.Timestamp timestamp = new java.sql.Timestamp(date.getTime());
//2017-08-27 22:18:23.857
```

## java.sql.Time
```java
java.util.Date date = new java.util.Date();
java.sql.Time time = new java.sql.Time(date.getTime());
//22:21:02
```