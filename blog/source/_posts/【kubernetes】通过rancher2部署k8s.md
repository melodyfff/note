---
title: 【kubernetes】通过rancher2部署k8s.md
tags: [kubernetes]
date: 2019-9-24
---

## K8S相关介绍

[十分钟带你理解Kubernetes核心概念](http://dockone.io/article/932)

## 部署rancher
```bash
# 更新操作系统软件包
yum update -y

# 删除历史容器及数据
docker rm -f $(docker ps -aq)
docker volume rm $(docker volume list -q)
rm -rf /var/lib/rancher /opt/cni /opt/containerd /opt/rke
systemctl stop firewalld
systemctl disable firewalld
systemctl daemon-reload
systemctl restart docker

# 创建 rancher
# /var/lib/rancher: etcd 数据存储
docker volume create rancher-data --label app=rancher
docker volume create rancher-log --label app=rancher
docker run -d --name=rancher-v2 --restart=unless-stopped \
    -p 8080:80 -p 8443:443 \
    -v rancher-data:/var/lib/rancher \
    -v rancher-log:/var/log/auditlog \
    -e AUDIT_LEVEL=3 \
    rancher/rancher:v2.2.8

# 定义 k8s 集群
# 基于 webconsole 进行 "Add Cluster", 只配置集群名称即可, 其它选默认配置

# 注册一个节点, 角色划分如下:
# - controlplane: master 节点, 即主控节点
# - etcd: etcd 节点, 即存储节点
# - worker: node 节点, 即计算节点
sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.2.8 --server ${IP} --token 8szs8xxgct5nmkl4dj9mqxpx4zrkf8c6j2blgb4jnj2kt5gp68cd9l --ca-checksum 14fc99cae672173b41e564f9389d1ef57dfcb5fdb1d204c615dcf01b5bb6575c --etcd --controlplane --worker
```