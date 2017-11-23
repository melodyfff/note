---
title: 【docker】SonarQube6.6后无法占用80端口问题
tags: [docker,linux,SonarQube]
date: 2017-11-23
---

近期由于需要对SonarQube进行升级到6.6以后的版本,发现6.6之后,sonar的`run.sh`不支持`root`用户启动,而是以`sonarqube`用户启动,导致1024以下的端口无法占用。  
因此特在原sonarqube镜像的基础上,加入nginx进行端口转发


## 制作sonarqube镜像

### 从官方镜像仓库下载

``` bash
$ docker pull sonarqube:6.7
```
### build新镜像加入nginx
将以下文件放置统一目录下
#### Dockerfile
```bash
FROM sonarqube:6.7
ENV SONAR_VERSION=6.7 \
    SONARQUBE_HOME=/opt/sonarqube 
MAINTAINER xinchen<xinchen@travelsky.com
ADD my.sh /bin/
ADD nginx.conf /bin/
RUN chmod +x /bin/my.sh
RUN /bin/my.sh
RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak
ADD sources.list /etc/apt/sources.list
RUN apt-get update
RUN apt-get install nginx
RUN mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
RUN mv /bin/nginx.conf /etc/nginx/


EXPOSE 80
EXPOSE 9000

WORKDIR $SONARQUBE_HOME
ENTRYPOINT ["./bin/run.sh"]
```

#### my.sh
```bash
# 加入代理 
sudo su -
cat << EOF > /etc/apt/apt.conf
Acquire::http::Proxy "http://172.x.x.x:8080";
Acquire::https::Proxy "http://172.x.x.x:8080";
Acquire::http::Proxy::172.x.x.x "DIRECT";
Acquire::http::Proxy::ftp.x.x "DIRECT";
EOF
```

#### sources.list
```bash
# sonar镜像由debian 打包
deb http://mirrors.cloud.aliyuncs.com/debian stable main contrib non-free
deb http://mirrors.cloud.aliyuncs.com/debian stable-proposed-updates main contrib non-free
deb http://mirrors.cloud.aliyuncs.com/debian stable-updates main contrib non-free
deb-src http://mirrors.cloud.aliyuncs.com/debian stable main contrib non-free
deb-src http://mirrors.cloud.aliyuncs.com/debian stable-proposed-updates main contrib non-free
deb-src http://mirrors.cloud.aliyuncs.com/debian stable-updates main contrib non-free

deb http://mirrors.aliyun.com/debian stable main contrib non-free
deb http://mirrors.aliyun.com/debian stable-proposed-updates main contrib non-free
deb http://mirrors.aliyun.com/debian stable-updates main contrib non-free
deb-src http://mirrors.aliyun.com/debian stable main contrib non-free
deb-src http://mirrors.aliyun.com/debian stable-proposed-updates main contrib non-free
deb-src http://mirrors.aliyun.com/debian stable-updates main contrib non-free
```
#### nginx.conf
```yml
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
        location / {
           proxy_pass   http://localhost:9000; 
        }
    }
}
```

## 部署

## 挂载本地

> 当 -v 绑定本地目录时
> 需要先 -v /opt/bin/run.sh:/opt/sonarqube/bin 运行启动脚本

run.sh
```sh
#!/bin/bash

set -e

if [ "${1:0:1}" != '-' ]; then
  exec "$@"
  # run nginx
  /usr/sbin/nginx
fi

export SONAR_VERSION=6.7
export SONARQUBE_HOME=/opt/sonarqube
export SONARQUBE_JDBC_USERNAME=
export SONARQUBE_JDBC_PASSWORD=
export SONARQUBE_JDBC_URL=

chown -R sonarqube:sonarqube $SONARQUBE_HOME
exec gosu sonarqube \
  java -jar lib/sonar-application-$SONAR_VERSION.jar \
  -Dsonar.log.console=true \
  -Dsonar.jdbc.username="$SONARQUBE_JDBC_USERNAME" \
  -Dsonar.jdbc.password="$SONARQUBE_JDBC_PASSWORD" \
  -Dsonar.jdbc.url="$SONARQUBE_JDBC_URL" \
  -Dsonar.web.javaAdditionalOpts="$SONARQUBE_WEB_JVM_OPTS -Djava.security.egd=file:/dev/./urandom" \
  -Dsonar.web.javaOpts="-Xmx512m -Xms256m -Djava.net.preferIPv4Stack=true -server" \
  -Dsonar.ce.javaOpts="-Xmx1536m -Xms512m -Djava.net.preferIPv4Stack=true -server" \
  -Dsonar.search.javaOpts="-Xmx512m -Xms256m -Xss1m -Djava.net.preferIPv4Stack=true" \
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

## nginx 常用命令
-  /usr/sbin/nginx -s stop  停止
-  /usr/sbin/nginx -s reload  重新加载
-  /usr/sbin/nginx -c /etc/nginx/nginx.conf  加载指定配置文件



