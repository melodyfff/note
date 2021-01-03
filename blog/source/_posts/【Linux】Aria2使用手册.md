---
title: 【Linux】Aria2使用手册
tags: [Linux]
date: 2021-01-03
---

GitHub地址: https://github.com/aria2/aria2
[aria2.conf - dht.dat](https://github.com/P3TERX/aria2.conf)
[aria2（命令行下载器）使用](https://www.jianshu.com/p/6e6a02e1f15e)
[磁力下载-trackerslist](https://github.com/ngosang/trackerslist)


## 使用实例

**指定配置文件**
```bash
aria2c --conf-path=/etc/aria2/aria2.conf 
```

**下载多个文件**

```bash
aria2c -Z $URL1 $URL2 ...
```

使用 `-P` 参数来扩展下载地址：

```bash
# 下载 1-68的文件，文件名以$FILE_NAME-1.jpg，$FILE_NAME-2.jpg...命名
aria2c -Z -P $URL/$FILE_NAME-{1..68}.jpg
```