---
title: 【Linux】VirtualBox网络配置桥接模式
tags: [Linux]
date: 2019-06-01
---

# VirtualBox网络配置桥接模式

## CentOS/RHEL (虚拟机)配置
```bash
# 基于桥接模式设置固定 ip
cat >> /etc/sysconfig/network-scripts/ifcfg-enp0s3 << EOF
# network setting
BOOTPROTO=static
# 固定ip
IPADDR=192.168.1.84
# 网关
GATEWAY=192.168.1.1
# DNS
DNS1=8.8.8.8
DNS2=114.114.114.114
EOF

# 重启network
service network restart
```
