---
title: 【docker】Docker快速构建Redis集群(cluster)
tags: [docker]
date: 2019-5-19
---

# Docker快速构建Redis集群(cluster)

以所有`redis`实例运行在同一台宿主机上为例子

## 搭建步骤

`redis`集群目录清单
```bash
.
├── Dockerfile
├── make_master_slave.sh
├── run_master_slave.sh
├── compose_master_slave.sh
├── redis-trib.rb
├── master
│   ├── 7000
│   │   ├── data
│   │   │   ├── appendonly.aof
│   │   │   ├── dump.rdb
│   │   │   └── nodes.conf
│   │   └── redis.conf
│   ├── 7001
│   │   ├── data
│   │   │   ├── appendonly.aof
│   │   │   ├── dump.rdb
│   │   │   └── nodes.conf
│   │   └── redis.conf
│   └── 7002
│       ├── data
│       │   ├── appendonly.aof
│       │   ├── dump.rdb
│       │   └── nodes.conf
│       └── redis.conf
├── redis-cluster.tmpl
└── slave
    ├── 7003
    │   ├── data
    │   │   ├── appendonly.aof
    │   │   ├── dump.rdb
    │   │   └── nodes.conf
    │   └── redis.conf
    ├── 7004
    │   ├── data
    │   │   ├── appendonly.aof
    │   │   ├── dump.rdb
    │   │   └── nodes.conf
    │   └── redis.conf
    └── 7005
        ├── data
        │   ├── appendonly.aof
        │   ├── dump.rdb
        │   └── nodes.conf
        └── redis.conf

```

### 1.redis.conf
找到一份原始的redis.conf文件，将其重命名为：redis-cluster.tmpl
**redis-cluster.tmpl**
```bash
# bind 127.0.0.1
protected-mode no
port ${PORT}
daemonize no
dir /data/redis
appendonly yes
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 15000
```

### 2.构建redis-trib镜像
`redis-trib.rb`是`redis`官方推出的管理`redis`集群的工具，集成在`redis`的源码`src`目录下,因为搭建redis-cluster的时候需要用到redis-trib工具。

构建`redis-trib`镜像
**Dockerfile**
```bash
FROM ruby:2.5.5-slim
MAINTAINER xinchen<xcmelody@gmail.com>
RUN gem install redis
RUN mkdir /redis
WORKDIR /redis
# redis-trib.rb 可在https://github.com/antirez/redis中找到
# 此处已经 wget https://raw.githubusercontent.com/antirez/redis/4.0/src/redis-trib.rb 
ADD ./redis-trib.rb /redis/redis-trib.rb
```

构建镜像
```bash
docker build -t redis-trib .
```

### 3.创建docker内部网络
```bash
# docker network ls 可查看
docker network create redis-cluster-net
```


### 4.创建 master 和 slave 文件夹并生成配置文件

**make_master_slave.sh**
```bash
# 创建 master 和 slave 文件夹
for port in `seq 7000 7005`; do
    ms="master"
    if [ $port -ge 7003 ]; then
        ms="slave"
    fi
    mkdir -p ./$ms/$port/ && mkdir -p ./$ms/$port/data \
    && PORT=$port envsubst < ./redis-cluster.tmpl > ./$ms/$port/redis.conf;
done
```

### 5.运行docker redis 的 master 和 slave 实例

**run_master_slave.sh**
```bash
# 运行docker redis 的 master 和 slave 实例
for port in `seq 7000 7005`; do
    ms="master"
    if [ $port -ge 7003 ]; then
        ms="slave"
    fi
    docker run -d -p $port:$port -p 1$port:1$port \
    -v $PWD/$ms/$port/redis.conf:/data/redis.conf \
    -v $PWD/$ms/$port/data:/data/redis \
    --restart always --name redis-$ms-$port --net redis-cluster-net \
    redis redis-server /data/redis.conf;
done
```

### 6.组装masters : slaves 节点参数

**组装masters : slaves 节点参数**
```bash
# 组装masters : slaves 节点参数
matches=""
for port in `seq 7000 7005`; do
    ms="master"
    if [ $port -ge 7003 ]; then
        ms="slave"
    fi
    matches=$matches$(docker inspect --format '{{(index .NetworkSettings.Networks "redis-cluster-net").IPAddress}}' "redis-$ms-${port}"):${port}" ";
done

echo $matches
# 172.20.0.2:7000 172.20.0.3:7001 172.20.0.4:7002 172.20.0.5:7003 172.20.0.6:7004 172.20.0.7:7005

# 创建docker-cluster，这里就用到了上面的redis-trib镜像
docker run -it --rm --net redis-cluster-net redis-trib ruby redis-trib.rb create --replicas 1 $matches

# 最后需要在接下来的console中输入“yes”，即可完成docker-cluster的搭建
```


## 参考

[docker redis 集群（cluster）搭建](https://my.oschina.net/dslcode/blog/1936656)

[redis官网](https://redis.io/documentation)

[redis中文文档](http://redis.cn/documentation.html)



