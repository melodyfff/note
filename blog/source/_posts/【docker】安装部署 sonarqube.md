---
title: 【docker】安装部署 sonarqube 
tags: [docker,linux]
date: 2017-07-21
---

近期由于项目需要，需要部署和维护[sonarqube](https://www.sonarqube.org)平台，运用[docker](http://docker.io/)进行部署，以下为部署步骤。

## 安装sonarqube镜像

### 从官方镜像仓库下载

``` bash
$ docker pull sonarqube:6.4
```

### 从阿里云仓库下载镜像

``` bash
$ docker pull registry.cn-hangzhou.aliyuncs.com/leansw/sonarqube
```

More info: [阿里云docker镜像](https://dev.aliyun.com/detail.html?spm=5176.1972343.2.14.599c57151UCJyX&repoId=10694u)

## 部署postgresql
官方运行指令
``` bash
$ docker run -d --name sonarqube \
-p 9000:9000 -p 9092:9092 \
-e SONARQUBE_JDBC_USERNAME=sonar \
-e SONARQUBE_JDBC_PASSWORD=sonar \
-e SONARQUBE_JDBC_URL=jdbc:postgresql://localhost/sonar sonarqube
```

> `--name` 重命名
> `-e`  绑定相关配置
> `-p`  映射端口
> `-d`  后台运行
> `sonarqube`  为镜像的版本号（或容器ID）

运行指令实例

```bash
docker run -d --name sonarqube \ 
-p 9000:9000 \ 
-e SONARQUBE_JDBC_USERNAME=sonar \ 
-e SONARQUBE_JDBC_PASSWORD= \ 
-e SONARQUBE_JDBC_URL=jdbc:postgresql://localhost/postgres sonar
```
## 挂载本地

> 当 -v 绑定本地目录时
> 需要先 -v /opt/bin/run.sh:/opt/sonarqube/bin 运行启动脚本

run.sh
```sh
#!/bin/bash

set -e

if [ "${1:0:1}" != '-' ]; then
  exec "$@"
fi

export TZ=Asia/Shanghai
export SONARQUBE_JDBC_USERNAME=test
export SONARQUBE_JDBC_PASSWORD=
export SONARQUBE_JDBC_URL=jdbc:postgresql://192.168.201.130:8090/sonar

exec java -jar lib/sonar-application-$SONAR_VERSION.jar \
  -Dsonar.jdbc.username="$SONARQUBE_JDBC_USERNAME" \
  -Dsonar.jdbc.password="$SONARQUBE_JDBC_PASSWORD" \
  -Dsonar.jdbc.url="$SONARQUBE_JDBC_URL" \
  -Dsonar.web.port=9000 \
  "$@"
```
运行实例
```bash
docker run -d --name sonarqube \ 
-p 9000:9000 \
-v /opt/sonarqube/sonar_bin/:/opt/sonarqube/bin \
-v /opt/sonarqube/sonar_plugs:/opt/sonarqube/extensions/plugins \ 
-v /opt/sonarqube/sonar_conf:/opt/sonarqube/conf 9a23ac22b32b
```


