---
title: 【Linux】zookeeper单机以及集群搭建
tags: [Linux]
date: 2019-5-11
---

# zookeeper单机以及集群搭建

> 下载地址: https://archive.apache.org/dist/zookeeper/

```bash
# 获取
wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz

# 解压
tar -zvxf zookeeper-3.4.14.tar.gz -C /usr/local/src/
ln -sv /usr/local/src/zookeeper-3.4.14/ /usr/local/zookeeper
cd /usr/local/zookeeper/
```

## 单机搭建

**新建配置 zoo.cfg**
```bash
# $zookeeper/conf 目录下新建 zoo.cfg
tickTime=2000
dataDir=/home/xinchen/zookeeper/data
clientPort=2181
initLimit=5
syncLimit=2
```

**启动**
```bash
# 启动
# 常用参数 zkServer.sh {start|start-foreground|stop|restart|status|upgrade|print-cmd}
$zookeeper/bin/zkServer.sh start

# 查看状态
ps -ef|grep zookeeper

# 查看端口状态
xinchen@ubuntu:~/zookeeper/zookeeper-3.4.12/bin$ echo stat| nc localhost 2181
Zookeeper version: 3.4.12-e5259e437540f349646870ea94dc2658c4e44b3b, built on 03/27/2018 03:55 GMT
Clients:
 /0:0:0:0:0:0:0:1:37374[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 1/22/44
Received: 3
Sent: 2
Connections: 1
Outstanding: 0
Zxid: 0x2
Mode: standalone
Node count: 4
```

**客户端连接**
```bash
$zookeeper/bin/zkCli.sh -server localhost:2181
```

## 集群配置

官网配置参考: https://zookeeper.apache.org/doc/r3.4.14/zookeeperAdmin.html

**创建配置文件**
```bash
cd $zookeeper
touch zoo1.cfg zoo2.cfg zoo3.cfg

# 主要模拟在单机上部署集群，具体多机部署可更改server ip即可
# 分别配置如下

# zoo1.cfg
tickTime=2000
dataDir=/home/xinchen/zookeeper/data/zoo1
clientPort=2181
initLimit=5
syncLimit=2
server.1=localhost:2886:3886
server.2=localhost:2887:3887
server.3=localhost:2888:3888

# zoo2.cfg
tickTime=2000
dataDir=/home/xinchen/zookeeper/data/zoo2
clientPort=2182
initLimit=5
syncLimit=2
server.1=localhost:2886:3886
server.2=localhost:2887:3887
server.3=localhost:2888:3888

# zoo3.cfg
tickTime=2000
dataDir=/home/xinchen/zookeeper/data/zoo3
clientPort=2183
initLimit=5
syncLimit=2
server.1=localhost:2886:3886
server.2=localhost:2887:3887
server.3=localhost:2888:3888
```

**配置数据目录**

```bash
cd /home/xinchen/zookeeper/data/
mkdir {zoo1,zoo2,zoo3}
echo 1 > zoo1/myid
echo 2 > zoo2/myid
echo 3 > zoo3/myid
```

**启动集群**
```bash
zkServer.sh start $zookeeper/conf/zoo1.cfg
zkServer.sh start $zookeeper/conf/zoo2.cfg
zkServer.sh start $zookeeper/conf/zoo3.cfg
```

**查看状态**
```bash
# 查看Mode 应该会有1个leader 2个follower
# echo stat|nc localhost 2181
# echo stat|nc localhost 2182
# echo stat|nc localhost 2183

xinchen@ubuntu:~/zookeeper/zookeeper-3.4.12/bin$ echo stat|nc localhost 2181
Zookeeper version: 3.4.12-e5259e437540f349646870ea94dc2658c4e44b3b, built on 03/27/2018 03:55 GMT
Clients:
 /0:0:0:0:0:0:0:1:37944[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 8/48/89
Received: 3
Sent: 2
Connections: 1
Outstanding: 0
Zxid: 0x100000001
Mode: follower
Node count: 4
xinchen@ubuntu:~/zookeeper/zookeeper-3.4.12/bin$ echo stat|nc localhost 2182
Zookeeper version: 3.4.12-e5259e437540f349646870ea94dc2658c4e44b3b, built on 03/27/2018 03:55 GMT
Clients:
 /0:0:0:0:0:0:0:1:54010[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/0
Received: 1
Sent: 0
Connections: 1
Outstanding: 0
Zxid: 0x100000001
Mode: leader
Node count: 4
xinchen@ubuntu:~/zookeeper/zookeeper-3.4.12/bin$ echo stat|nc localhost 2183
Zookeeper version: 3.4.12-e5259e437540f349646870ea94dc2658c4e44b3b, built on 03/27/2018 03:55 GMT
Clients:
 /0:0:0:0:0:0:0:1:35350[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/0
Received: 1
Sent: 0
Connections: 1
Outstanding: 0
Zxid: 0x100000001
Mode: follower
Node count: 4

```

**客户端连接**
```bash
zkCli.sh -server localhost:2181,localhost:2182,localhost:2183
```

