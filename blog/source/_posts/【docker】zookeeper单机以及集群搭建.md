---
title: 【docker】zookeeper单机以及集群搭建
tags: [docker]
date: 2019-5-10
---

# zookeeper单机以及集群搭建

> [HUB ZOOKEEPER](https://hub.docker.com/_/zookeeper)


## ZK单机
```bash
# 此镜像默认暴露的端口为 ：2181 2888 3888
# 启动单机zk
> docker run --name some-zookeeper -p 2181:2181 --restart always -d zookeeper:3.4.14
# 挂载配置卷
# docker run --name some-zookeeper -p 2181:2181 --restart always -d -v $(pwd)/zoo.cfg:/conf/zoo.cfg zookeeper:3.4.14

# 连接到zk
> docker run -ti --rm --link some-zookeeper:zookeeper zookeeper:3.4.14 zkCli.sh -server zookeeper

```

## ZK集群

#### docker-compose.yml
```yml

# ZOO_MY_ID表示ZK服务的 ID，它是1-255 之间的整数，必须在集群中唯一
# ZOO_SERVERS是ZK 集群的主机列表

version: '3.1'

services:
  zoo1:
    image: zookeeper:3.4.14
    restart: always
    hostname: zoo1
    # container_name: zoo1
    ports:
      - 2181:2181
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=0.0.0.0:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888

  zoo2:
    image: zookeeper:3.4.14
    restart: always
    hostname: zoo2
    # container_name: zoo2
    ports:
      - 2182:2181
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=0.0.0.0:2888:3888 server.3=zoo3:2888:3888

  zoo3:
    image: zookeeper:3.4.14
    restart: always
    hostname: zoo3
    # container_name: zoo3
    ports:
      - 2183:2181
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=0.0.0.0:2888:3888
```


```bash
# 基于docker-compose
# docker-compose的安装可参考https://docs.docker.com/compose/install 
# 不同版本的区别可参考https://github.com/docker/docker.github.io/blob/master/compose/compose-file/compose-versioning.md
# 目前使用的版本为 1.24.0


# 启动命令
# COMPOSE_PROJECT_NAME=zk_test 这个环境变量, 这是为我们的 compose 工程起一个名字, 以免与其他的 compose 混淆
> COMPOSE_PROJECT_NAME=zk_test docker-compose -f docker-compose.yml up -d


# 查询状态
> COMPOSE_PROJECT_NAME=zk_test docker-compose ps
#     Name                   Command               State                     Ports                   
#----------------------------------------------------------------------------------------------------
#zk_test_zoo1_1   /docker-entrypoint.sh zkSe ...   Up      0.0.0.0:2181->2181/tcp, 2888/tcp, 3888/tcp
#zk_test_zoo2_1   /docker-entrypoint.sh zkSe ...   Up      0.0.0.0:2182->2181/tcp, 2888/tcp, 3888/tcp
#zk_test_zoo3_1   /docker-entrypoint.sh zkSe ...   Up      0.0.0.0:2183->2181/tcp, 2888/tcp, 3888/tcp
```

#### 本地连接
```bash
> zkCli.sh -server localhost:2181,localhost:2182,localhost:2183
```

#### 查看集群状态
```bash
# echo stat | nc localhost 2181
# echo stat | nc localhost 2182
# echo stat | nc localhost 2183
# 观察Mode 此时应该具有  leader follower follower


xinchen@ubuntu:~/zk$ echo stat | nc localhost 2181
Zookeeper version: 3.4.14-4c25d480e66aadd371de8bd2fd8da255ac140bcf, built on 03/06/2019 16:18 GMT
Clients:
 /172.19.0.1:51814[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/0
Received: 1
Sent: 0
Connections: 1
Outstanding: 0
Zxid: 0x100000000
Mode: follower
Node count: 4


xinchen@ubuntu:~/zk$ echo stat | nc localhost 2182
Zookeeper version: 3.4.14-4c25d480e66aadd371de8bd2fd8da255ac140bcf, built on 03/06/2019 16:18 GMT
Clients:
 /172.19.0.1:46810[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/0
Received: 1
Sent: 0
Connections: 1
Outstanding: 0
Zxid: 0x0
Mode: follower
Node count: 4


xinchen@ubuntu:~/zk$ echo stat | nc localhost 2183
Zookeeper version: 3.4.14-4c25d480e66aadd371de8bd2fd8da255ac140bcf, built on 03/06/2019 16:18 GMT
Clients:
 /172.19.0.1:43796[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/0
Received: 1
Sent: 0
Connections: 1
Outstanding: 0
Zxid: 0x100000000
Mode: leader
Node count: 4
Proposal sizes last/min/max: -1/-1/-1

```