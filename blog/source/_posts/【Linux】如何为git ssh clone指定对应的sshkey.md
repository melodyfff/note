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

> 注： 在对应项目添加.gitconfig文件也可直接影响git配置