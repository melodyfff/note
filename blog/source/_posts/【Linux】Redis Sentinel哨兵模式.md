---
title: 【Linux】Redis Sentinel哨兵模式
tags: [Linux]
date: 2020-1-14
---

# Redis Sentinel 哨兵模式
Redis Sentinel provides high `availability` for Redis.

- **监控:** Sentinel会不断检查您的主实例master和副本实例slave是否按预期工作。
- **通知:** Sentinel可以通过API通知系统管理员或其他计算机程序，其中一个受监视的Redis实例出了问题。
- **自动故障转移(failover):** 如果主服务器master未按预期工作，则Sentinel可以启动故障转移过程，在该过程中，将副本slave升级为主服务器，将其他附加副本重新配置为使用新的主服务器，并通知使用Redis服务器的应用程序连接时要使用的新地址。
- **配置提供程序:** Sentinel充当客户端服务发现的授权来源：客户端连接到Sentinels，以询问负责给定服务的当前Redis主服务器的地址。如果发生故障转移，Sentinels将报告新地址。

## 有关Sentinel的基本知识
- 一个健壮的部署至少需要三个Sentinel实例,并且这三个Sentinel应该部署在独立的服务器或虚拟机上。
- 由于Redis使用异步复制，因此Sentinel + Redis分布式系统不能保证在故障期间保留已确认的写入(一致性问题)。

### Redis-Sentinel 为什么不推荐使用两台Sentinel

在 sentinel 启动故障转移（failover）时需要满足两个条件：
- 确定 master 不可用的 sentinel 数量必须大于等于 仲裁 (quorum)
- 大多数 (majority) 的 sentinel 之间必须可以通信（大多数的意思是两台就是2，三台也是2，四台是2,五台就是3）这里通信目的是选出谁来执行 failover 操作

### Redis 哨兵主备切换的数据丢失问题
主备切换的过程，可能会导致数据丢失:

#### 异步复制导致的数据丢失
因为 master->slave 的复制是异步的，所以可能有部分数据还没复制到 slave，master 就宕机了，此时这部分数据就丢失了。
```bash
                                         +-------------+
                                         |             |
                                         |    Master   |
                                         |             |
                                         +-----+-------+
                                               ^
                                               |
                                               | elect to master
                                               |
+----------------+                 +-----------+-----+
|                |                 |                 |
|      Master    +----------------->       Slave     |
|                |                 |                 |
+--------+-------+                 +----------+------+
         |                                    ^
         |                                    |
         |                                    |
         +                                    |
Data prepare copy,                      +-----+
master down                             |
                                        |
                            +-----------+------------+
                            |                        |
                            |        Sentinels       |
                            |                        |
                            +------------------------+

```
#### 脑裂导致的数据丢失
脑裂，也就是说，某个 master 所在机器突然脱离了正常的网络，跟其他 slave 机器不能连接，但是实际上 master 还运行着。此时哨兵可能就会认为 master 宕机了，然后开启选举，将其他 slave 切换成了 master。这个时候，集群里就会有两个 master ，也就是所谓的脑裂。

此时虽然某个 slave 被切换成了 master，但是可能 client 还没来得及切换到新的 master，还继续向旧 master 写数据。因此旧 master 再次恢复的时候，会被作为一个 slave 挂到新的 master 上去，自己的数据会清空，重新从新的 master 复制数据。而新的 master 并没有后来 client 写入的数据，因此，这部分数据也就丢失了。

#### 数据丢失问题解决
对`redis.conf`进行如下配置
```bash
min-slaves-to-write 1
min-slaves-max-lag 10
```
表示，要求至少有 1 个 slave，数据复制和同步的延迟不能超过 10 秒。

如果说一旦所有的 slave，数据复制和同步的延迟都超过了 10 秒钟，那么这个时候，master 就不会再接收任何请求了。

如果一个 master 出现了脑裂，跟其他 slave 丢了连接，那么上面两个配置可以确保说，如果不能继续给指定数量的 slave 发送数据，而且 slave 超过 10 秒没有给自己 ack 消息，那么就直接拒绝客户端的写请求。因此在脑裂场景下，最多就丢失 10 秒的数据。

### sdown 和 odown 转换机制
- sdown 是主观宕机，就一个哨兵如果自己觉得一个 master 宕机了，那么就是主观宕机
- odown 是客观宕机，如果 quorum 数量的哨兵都觉得一个 master 宕机了，那么就是客观宕机

### slave->master 选举算法
如果一个 master 被认为 odown 了，而且 majority 数量的哨兵都允许主备切换，那么某个哨兵就会执行主备切换操作，此时首先要选举一个 slave 来，会考虑 slave 的一些信息：

- 跟 master 断开连接的时长
- slave 优先级
- 复制 offset
- run id

如果一个 slave 跟 master 断开连接的时间已经超过了 down-after-milliseconds 的 10 倍，外加 master 宕机的时长，那么 slave 就被认为不适合选举为 master。
```bash
(down-after-milliseconds * 10) + milliseconds_since_master_is_in_SDOWN_state
```

接下来会对 slave 进行排序：
- 按照 slave 优先级进行排序，slave priority 越低，优先级就越高。
- 如果 slave priority 相同，那么看 replica offset，哪个 slave 复制了越多的数据，offset 越靠后，优先级就越高。
- 如果上面两个条件都相同，那么选择一个 run id 比较小的那个 slave。

### quorum 和 majority
每次一个哨兵要做主备切换，首先需要 quorum 数量的哨兵认为 odown，然后选举出一个哨兵来做切换，这个哨兵还需要得到 majority 哨兵的授权，才能正式执行切换。

如果 quorum < majority，比如 5 个哨兵，majority 就是 3，quorum 设置为 2，那么就 3 个哨兵授权就可以执行切换。

但是如果 quorum >= majority，那么必须 quorum 数量的哨兵都授权，比如 5 个哨兵，quorum 是 5，那么必须 5 个哨兵都同意授权，才能执行切换。

## 配置Sentinel

Sentinels默认情况下监听TCP连接端口号为26379

**默认启动方式**
```bash
# 以下两种方式相同
# 在运行Sentinel时必须使用配置文件，因为系统将使用此文件来保存当前状态，以便在重启时重新加载。如果未提供配置文件或配置文件路径不可写，Sentinel只会拒绝启动。
redis-sentinel /path/to/sentinel.conf
redis-server /path/to/sentinel.conf --sentinel
```

**官方典型最小配置**
```bash
# 告诉Redis监视一个名为 mymaster的主服务器
# 服务器地址为127.0.0.1 端口为 6379 仲裁quorum为2
sentinel monitor mymaster 127.0.0.1 6379 2

# master被认为s_down的时间,单位毫秒 默认30秒
sentinel down-after-milliseconds mymaster 60000

# 指定故障转移超时（以毫秒为单位）,默认3分钟
# 指定当故障发生的时候，进行恢复的超时时间，当failover开始后，在此时间内仍然没有触发任何failover操作，当前sentinel  将会认为此次failoer失败
sentinel failover-timeout mymaster 180000

# 在故障转移期间，多少个副本节点进行数据同步
sentinel parallel-syncs mymaster 1
```

## Docker Compose部署
```bash
git clone https://github.com/AliyunContainerService/redis-cluster
cd redis-cluster
```
目录结构为
```bash
.
├── docker-compose.yml
├── LICENSE
├── README.md
├── sentinel
│   ├── Dockerfile
│   ├── sentinel.conf
│   └── sentinel-entrypoint.sh
└── test.sh
```

修改`sentinel`目录下`Dockerfile`中redis的版本号,并构建镜像
```bash
# 我使用的是redis-3.2.8,为方便使用,后续镜像命名为redis:3
docker build -t redis:3 .
```

添加到镜像中的`sentinel.conf`配置如下:
```bash
# redis-master为docker网络中的--link别名
sentinel monitor mymaster redis-master 6379 2
sentinel down-after-milliseconds mymaster 5000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 5000
```

`docker-compose.yml`中的配置为如下:
```yaml
master:
  image: redis:3.2.8
slave:
  image: redis:3.2.8
  command: redis-server --slaveof redis-master 6379
  links:
    # master别名redis-master
    - master:redis-master
sentinel:
  # 这里指定构建镜像地址
  build: sentinel
  environment:
    - SENTINEL_DOWN_AFTER=5000
    - SENTINEL_FAILOVER=5000    
  links:
    - master:redis-master
    - slave
```

在模板中定义了下面一系列服务
- master: Redis master
- slave: Redis slave
- sentinel: Redis Sentinel

启动
```bash
# 这里会自动去构建sentinel文件夹下的镜像
# 也可以先docker-compose build
docker-compose up -d
```

查看启动情况
```bash
$docker-compose ps

          Name                        Command               State          Ports       
---------------------------------------------------------------------------------------
redis-cluster_master_1     docker-entrypoint.sh redis ...   Up      6379/tcp           
redis-cluster_sentinel_1   sentinel-entrypoint.sh           Up      26379/tcp, 6379/tcp
redis-cluster_slave_1      docker-entrypoint.sh redis ...   Up      6379/tcp   
```

伸缩sentinel和slave实例
```bash
docker-compose scale sentinel=3
docker-compose scale slave=2
```

再次查看集群状态
```bash
$docker-compose ps

         Name                        Command               State          Ports       
---------------------------------------------------------------------------------------
redis-cluster_master_1     docker-entrypoint.sh redis ...   Up      6379/tcp           
redis-cluster_sentinel_1   sentinel-entrypoint.sh           Up      26379/tcp, 6379/tcp
redis-cluster_sentinel_2   sentinel-entrypoint.sh           Up      26379/tcp, 6379/tcp
redis-cluster_sentinel_3   sentinel-entrypoint.sh           Up      26379/tcp, 6379/tcp
redis-cluster_slave_1      docker-entrypoint.sh redis ...   Up      6379/tcp           
redis-cluster_slave_2      docker-entrypoint.sh redis ...   Up      6379/tcp     
```

执行测试脚本`./test.sh`

这个测试脚本实际上利用 docker pause 命令将 Redis master容器暂停，sentinel会发现这个故障并将master切换到其他一个备用的slave上面。

```bash
Redis master: 172.172.172.2
Redis Slave: 172.172.172.3
------------------------------------------------
Initial status of sentinel
------------------------------------------------
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=172.172.172.2:6379,slaves=2,sentinels=3
Current master is
172.172.172.2
6379
------------------------------------------------
Stop redis master
redis-cluster_master_1
Wait for 10 seconds
Current infomation of sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=172.172.172.3:6379,slaves=2,sentinels=3
Current master is
172.172.172.3
6379
------------------------------------------------
Restart Redis master
redis-cluster_master_1
Current infomation of sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=172.172.172.3:6379,slaves=2,sentinels=3
Current master is
172.172.172.3
6379
```

停止与清理
```bash
docker-compose stop
docker-compose rm
```

## 参考
[Redis Sentinel Documentation](https://redis.io/topics/sentinel)

[sentinel.conf](http://download.redis.io/redis-stable/sentinel.conf)

[Redis 哨兵集群实现高可用](https://github.com/doocs/advanced-java/blob/master/docs/high-concurrency/redis-sentinel.md)

[使用Docker Compose部署基于Sentinel的高可用Redis集群](https://yq.aliyun.com/articles/57953)
