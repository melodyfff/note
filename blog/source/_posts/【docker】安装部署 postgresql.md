---
title: 【docker】安装部署 postgresql
tags: [docker,linux]
---

近期由于项目需要，需要部署和维护[sonarqube](https://www.sonarqube.org)平台，运用[docker](http://docker.io/)进行部署，以下为部署步骤。

## 安装postgresql镜像

### 从官方镜像仓库下载

``` bash
$ docker pull postgresql:9.6
```

### 从阿里云仓库下载镜像

``` bash
$ docker pull registry.cn-hangzhou.aliyuncs.com/lsghst/postgresql
```

More info: [阿里云docker镜像](https://dev.aliyun.com/list.html?namePrefix=ubuntu)

## 部署postgresql

``` bash
$ sudo docker run --name pg_test \
-e POSTGRES_PASSWORD=123 \
-e POSTGRES_USER=sonar \
-e POSTGRES_DB=sonar \
-p 8090:5432 -d postgres:9.6
```

> `--name` 重命名
> `-e`  绑定相关配置
> `-p`  映射端口
> `-d`  后台运行
> `postgres:9.6`  为镜像的版本号（或容器ID）

## 访问postgres容器

### 外部访问

```bash
psql -h localhost -p 8090 -d sonar -U sonar --password
```
> `localhost`为虚拟机IP地址,docker的IP地址为内部通讯使用

### 访问容器内部

```bash
sudo docker exec -it 3bf3b533ba05 /bin/bash
```

## 挂载数据

### 挂载宿主机

```bash
sudo docker run --name ps_test -d \
-v /opt/data:/var/lib/postgresql/data \
-p 8090:5432 \ 
-e POSTGRES_USER=sonar \ 
-e POSTGRES_DB=sonar  0f3af79d8673
```

### 挂载数据卷容器
通常使用数据容器来持久化数据库和数据文件
```bash
docker run --name dbdata -d \ 
-v /opt/data:/var/lib/postgresql/data \
postgres:9.6 echo "data only"
```
创建了一个名为dbdata的数据容器，运行完echo之后就停止了。数据容器是不需要运行的，只要创建好了就可以了。

```bash
sudo docker run -d --volumes-from dbdata \ 
--name pgtest \ 
-p 8090:5432 \ 
-e POSTGRES_USER=sonar \ 
-e POSTGRES_DB=postgres postgres:9.6
```

## 备份、恢复或迁移volume
```bash
docker run --rm --volumes-from dbdata \ 
-v $(pwd):/backup 0f3af79d8673 \ 
tar cvf /backup/pg_test.tar.gz /var/lib/postgresql/data
```

## 查看容器信息
```bash
docker inspect f1c87508fe97
```

## 容器内外互相拷贝数据
```bash
docker cp foo.txt mycontainer:/foo.txt
docker cp mycontainer:/foo.txt foo.txt
```