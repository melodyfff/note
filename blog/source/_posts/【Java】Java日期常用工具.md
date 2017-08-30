---
title: 【Java】Java 日期常用工具
tags: [java]
date: 2017-8-27
---

## 获取日期之间的差值（天）
```java
private int getDiffDay(Date dateStar,Date dateEnd){
long diffTime = dateStar.getTime() - dateEnd.getTime();//差值微秒级别
return (int) (diffTime/(1000 * 60 * 60 * 24));
//2004-01-03 23:59:59 - 2004-01-02 0:0:0 = 1
    }
```