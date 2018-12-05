---
title: 【docker】docker redis 使用
tags: [docker]
date: 2018-12-5
---

> 参考： https://redis.io/topics/persistence  
> 镜像库： https://hub.docker.com/_/redis

## 1.Redis以RDB存储启动

#### redis.conf
```bash
save 900 1
save 300 10
save 60 10000
dbfilename hello.rdb
dir /data
requirepass yourpass
```

#### 运行
```bash
docker run --name redis \
 -v ~/conf/redis.conf:/usr/local/etc/redis/redis.conf \
 -v ~/data:/data -d -p 6379:6379 redis:4.0.11 \
  redis-server  /usr/local/etc/redis/redis.conf
```

## 2.Redis以AOF存储启动

#### 运行
```bash
docker run --name redis \
 -v ~/data:/data -d -p 6379:6379 redis:4.0.11 \
  redis-server \
  --appendonly yes \
  --requirepass 'password'
```

