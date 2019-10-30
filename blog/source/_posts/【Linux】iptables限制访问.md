---
title: 【Linux】iptables限制访问
tags: [Linux]
date: 2019-10-30
---
# iptables限制访问
## 常用命令
```bash
# 查看规则
iptables -L INPUT --line-numbers

# 开放指定的端口
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# 禁止指定端口
iptables -A INPUT -p tcp --dport 80 -j DROP

# 拒绝所有端口
iptables -A INPUT -j DROP
```
---
## 限制ip
```bash
# 限制单个ip的所有端口访问
iptables -I INPUT -s 211.1.0.1 -j DROP

# 封IP段所有端口访问
iptables -I INPUT -s 211.1.0.0/16 -j DROP
iptables -I INPUT -s 211.2.0.0/16 -j DROP
iptables -I INPUT -s 211.3.0.0/16 -j DROP

# 封整个段所有端口访问
iptables -I INPUT -s 211.0.0.0/8 -j DROP
```
---
## 限制端口
```bash
# 限制9889端口 tcp访问
iptables -I INPUT -p tcp --dport 9889 -j DROP

# 允许211.1.0.1 tcp访问9889端口
iptables -I INPUT -s 211.1.0.1 -p tcp --dport 9889 -j ACCEPT

# 限制211.1.0.1 tcp访问9889端口
iptables -I INPUT -s 211.1.0.1 -p tcp --dport 9889 -j DROP
```
---
## 限制并发访问
```bash
# 限制211.1.0.1访问80端口的并发数不超过10
iptables -I INPUT -p tcp --dport 80 -s 211.1.0.1 -m connlimit --connlimit-above 10 -j REJECT
```
---
## 解除封印
```bash
# 解除所有
iptables -L INPUT

# 解除单个(iptables -L --line-numbers 查看id)
iptables -D INPUT $ID
```