---
title: 【docker】docker代理配置
tags: [docker]
date: 2018-2-6
---

# Docker代理启动参数配置

```bash

mkdir -p /etc/systemd/system/docker.service.d

#网络代理配置
cat << EOF > /etc/systemd/system/docker.service.d/proxy.conf
[Service]
Environment="HTTP_PROXY=http://${ip}:${port}"
Environment="HTTPS_PROXY=http://${ip}:${port}"
Environment="NO_PROXY=127.0.0.1,localhost"
EOF

#启动参数配置（包括 docker 镜像代理配置）
cat << EOF > /etc/docker/daemon.json
{
    # 指定路径
    "graph": "/opt/lib/docker",
    "live-restore": true,
    "bip": "172.127.127.1/24",
    # dns解析地址
    "dns": ["${ip}"],
    # 指定仓库
    "registry-mirrors": ["${registry-address}"],
    # 信任   
    "insecure-registries": ["", "", ""]
}
EOF

#将当前用户加入到 docker 组，以便可以使用当前用户执行 docker 指令
exit
sudo addgroup docker
sudo usermod -aG docker $USER 

#刷新配置并重启 docker
sudo systemctl daemon-reload
sudo systemctl restart docker

```