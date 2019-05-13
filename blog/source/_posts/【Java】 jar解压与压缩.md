---
title: 【Java】 jar解压与压缩
tags: [java]
date: 2018-9-29
---


## jar解压与压缩
命令格式:`jar {c t x u f }[ v m e 0 M i ][-C 目录]文件名` 
```bash
# 解压,到当前目录
jar -xvf source.jar 

# 打包,不进行压缩
jar -cvfM0 source.jar ./
```