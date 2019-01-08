---
title: 【docker】docker alpine 镜像初始化
tags: [docker]
date: 2019-01-08
---

> [alpine](https://hub.docker.com/_/alpine)

## 构建初始化镜像
```dockerfile
FROM alpine:3.8
MAINTAINER xcmelody@qq.com
# 更换阿里源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories
# 如果在proxy环境下
# RUN HTTP_PROXY=http://$IP:$PORT apk add --no-cache vim
run apk update
run apk add --no-cache vim
run apk add --no-cache openssh

EXPOSE 22
```


