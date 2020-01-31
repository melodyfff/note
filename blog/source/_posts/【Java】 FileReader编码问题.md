---
title: 【Java】 FileReader编码问题
tags: [java]
date: 2020-1-31
---

# FileReader编码问题

在一次使用`FileReader`读取文件转换`json`时，出现了在`linux`上正常在`windows`上转换错误的情况

因为`FileReader`使用的是系统默认字符集去读取

```java
new FileReader(new File(x)).getEncoding() // windows上为GBK
```


解决方式为：
```java
new InputStreamReader(new FileInputStream(new File(x)), StandardCharsets.UTF_8) // 指定读取字符集
```