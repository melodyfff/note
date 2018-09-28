---
title: 【Linux】常用网络命令和工具
tags: [Linux]
date: 2018-09-28
---

## 常用端口占用查询

参考：http://int32bit.me/2016/05/04/Linux常用网络工具总结/

```bash

# 查看linux端口占用情况
netstat -tln

# 用于显示tcp，udp的端口和进程
netstat -tunlp | grep $PORT

# 用于查看某一端口的占用情况
lsof -i:80
```