---
title: 【Linux】Redis cluster集群模式
tags: [Linux]
date: 2020-1-15
---

# Redis cluster集群模式

> Redis Cluster设计的核心思想：数据拆分、去中心化

## Redis cluster基础概念

### Redis群集TCP端口
每个Redis群集节点都需要打开两个TCP连接。用于服务客户端的常规Redis TCP端口，例如6379，再加上将数据端口加10000所获得的端口，因此在示例中为16379。

第二个高端口用于群集总线，即使用二进制协议的节点到节点通信通道。节点将群集总线用于故障检测，配置更新，故障转移授权等。

### Redis Cluster data sharding
Redis Cluster将所有数据按照hash slot算法分布到16384[0-16383]个哈希槽上面，哈希槽分布在各节点上，各节点维护自己的哈希槽。

hash slot算法如下:
```bash
HASH_SLOT = CRC16(key) mod 16384
```

### Redis Cluster主从模型
为了在主节点的子集出现故障或无法与大多数节点通信时保持可用，Redis Cluster使用主从模型，其中每个哈希槽具有从1（主节点本身）到N个副本（N个） -1个其他从属节点）。

在具有节点A，B，C的示例集群中，如果节点B失败，则集群将无法继续，因为我们不再有办法为5501-11000范围内的哈希槽提供服务。

但是，在创建集群（或稍后）时，我们向每个主节点添加一个从属节点，以便最终集群由作为主节点的A，B，C和作为从属节点的A1，B1，C1组成，如果节点B发生故障，系统将能够继续。

节点B1复制B，并且B发生故障，群集会将节点B1提升为新的主节点，并将继续正常运行。

但是请注意，如果节点B和B1同时失败，则Redis Cluster无法继续运行。

### Redis集群一致性保证
Redis Cluster无法保证强一致性。实际上，这意味着在某些情况下，Redis Cluster可能会丢失系统认可给客户端的写入。

Redis Cluster可能丢失写入的第一个原因是因为它使用异步复制。这意味着在写入期间会发生以下情况：

- 您的客户写信给主B。
- 主B向您的客户答复OK。
- 主机B将写操作传播到其从机B1，B2和B3。

如您所见，B在回复客户端之前不会等待B1，B2，B3的确认，因为这会对Redis造成延迟，因此，如果您的客户端写了一些东西，B会确认写，但是在能够将写操作发送到其从属服务器之前崩溃，因此一个从属服务器（未接收到写操作）可以升级为主服务器，从而永远丢失该写操作。

### Redis cluster和Redis Sentinel的区别
`Redis cluster:`

- 是为了解决单机Redis容量有限的问题，将数据按一定的规则分配到多台机器,内存/QPS不受限于单机，可受益于分布式集群高扩展性。
- cluster做了sharding，一个key通过hash算法分配到不同的槽(slot)上，所有不同节点存储的数据是不一样的。
- 集群模式提高并发量

`Redis Sentinel:`
- 高可用性(HA)解决方案

## Dokcer部署
### 安装
创建工作目录,后面所有操作均在此目录下完成
```bash
mkdir cluster-test && cd cluster-test
```

准备镜像和网络
```bash
# 镜像
docker pull redis:4.0.14
docker pull ruby

# 网络
docker network create redis-net
# 查看虚拟网卡网关ip
docker network inspect redis-net | grep "Gateway" | grep --color=auto -P '(\d{1,3}.){3}\d{1,3}' -o
```

`redis-trib`帮助镜像准备
**Dockerfile**
```bash
FROM ruby
WORKDIR /redis
RUN gem install redis
# RUN wget http://download.redis.io/redis-stable/src/redis-trib.rb -P /redis
# 这里需要注意版本问题,5.x新版本不推荐使用
RUN wget https://raw.githubusercontent.com/antirez/redis/4.0/src/redis-trib.rb -P /redis
```

初始化脚本
```bash
export GATE_IP=$(docker network inspect redis-net | grep "Gateway" | grep --color=auto -P '(\d{1,3}.){3}\d{1,3}' -o)

cat > redis.tmpl << EOF
##节点端口
port \${PORT}

##cluster集群模式 
cluster-enabled yes
##集群配置名
cluster-config-file nodes.conf
##超时时间
cluster-node-timeout 5000
##实际为各节点网卡分配ip  先用上网关ip代替
cluster-announce-ip $GATE_IP
##节点映射端口
cluster-announce-port \${PORT}
##节点总线端
cluster-announce-bus-port 1\${PORT}
##持久化模式
appendonly yes
EOF

echo "======================================"
echo "Redis Cluster Cleaner."
echo "======================================"
# 清空之前创建的集群
for i in `seq 7000 7005`;do docker rm -f redis-$i;done

echo "======================================"
echo "Redis Cluster Conf Init."
echo "======================================"
for port in `seq 7000 7005`; do \
mkdir -p ./${port} \
&& mkdir -p ./${port}/data \
&& PORT=${port} envsubst < ./redis.tmpl > ./${port}/redis.conf; \
done

echo "======================================"
echo "Redis Cluster Start."
echo "======================================"
for port in `seq 7000 7005`; do \
docker run -d -ti -p ${port}:${port} -p 1${port}:1${port} \
-v $PWD/${port}/redis.conf:/usr/local/etc/redis/redis.conf \
-v $PWD/${port}/data:/data \
--restart always --name redis-${port} --net redis-net \
--sysctl net.core.somaxconn=1024 redis:4.0.14 redis-server /usr/local/etc/redis/redis.conf; \
done


echo "======================================"
echo "Redis Cluster Conf Replace."
echo "======================================"
matches=""
for port in `seq 7000 7005`; do \
hello=$(docker inspect redis-$port|grep "IPAddress"|grep --color=auto -P '(\d{1,3}.){3}\d{1,3}' -o) &&\
sed -i "s/$GATE_IP/$hello/g" ${port}/redis.conf &&\
docker restart redis-${port}

matches=$matches" "$hello":"$port
done

echo $matches

echo "======================================"
echo "Redis Cluster Start Up."
echo "======================================"
# docker run -it --rm --net redis-net redis:4.0.14  redis-cli --cluster create $matches --cluster-replicas 1
docker run -it --rm --net redis-net redis-trib ruby redis-trib.rb create --replicas 1 $matches
```

### 测试
```bash
# 这里一定要加-c启动集群模式连接
$redis-cli -c -h 127.0.0.1 -p 7000

# 可以发现来回切换
127.0.0.1:7000> set a a
-> Redirected to slot [15495] located at 172.18.0.4:7002
OK
172.18.0.4:7002> set bb bb
-> Redirected to slot [8620] located at 172.18.0.3:7001
OK
172.18.0.3:7001> set cc cc
-> Redirected to slot [700] located at 172.18.0.2:7000
OK
172.18.0.2:7000> 
```

## 参考
[Redis cluster tutorial](https://redis.io/topics/cluster-tutorial)

[Redis Cluster Specification](https://redis.io/topics/cluster-spec#keys-distribution-model)

[【译】Redis集群规范 (Redis Cluster Specification)](https://www.jianshu.com/p/8a2d810402a9)

[redis.conf](http://download.redis.io/redis-stable/redis.conf)

[Redis的高可用详解：Redis哨兵、复制、集群的设计原理，以及区别](https://youzhixueyuan.com/redis-high-availability.html)

[Redis Cluster 原理分析](https://www.jianshu.com/p/0232236688c1)

[docker redis 集群（cluster）搭建](https://my.oschina.net/dslcode/blog/1936656)
