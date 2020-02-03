---
title: 【Linux】如何为git ssh clone指定对应的sshkey
tags: [Linux]
date: 2020-02-03
---

## 配置.ssh/config

```bash
Host my
    HostName github.com
    User username
    PreferredAuthentications publickey
    IdentityFile ~/some/id_rsa
```

进行`clone`的时候
```bash
# 原地址为 ： ssh://git@github.com/some.git
git clone ssh://git@my:80/some.git
```

## 单个项目的git配置
在git中，我们使用git config 命令用来配置git的配置文件，git配置级别主要有以下3类：
- 1、仓库级别 local 【优先级最高】
- 2、用户级别 global【优先级次之】
- 3、系统级别 system【优先级最低】

```bash
git config --local -l 查看仓库配置
git config --global -l 查看用户配置
git config --system -l 查看系统配置
```