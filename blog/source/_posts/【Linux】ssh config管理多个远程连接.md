---
title: 【Linux】ssh config管理多个远程连接
tags: [Linux]
date: 2018-10-06
---

## 1.配置文件说明
```bash
# 用户配置文件
$ ~/.ssh/config
# 系统配置文件
$ /etc/ssh/ssh_config
# 本地生成公匙
$ ssh-keygen -t rsa
# 将公匙拷贝到服务器上
ssh-copy-id ~/.ssh/id_rsa.pub xinchen@192.168.0.222
scp ~/.ssh/id_rsa.pub xinchen@192.168.0.222:～/.ssh/authorized_keys
```

## 2.配置文件格式
```bash
Host firsthost
    # 主机地址
    HostName deepzz.com
    # 用户名
    User xinchen
    # 认证文件                   
    IdentityFile ~/.ssh/id_ecdsa
    # 指定端口       
    Port 22                      
    SSH_OPTIONS_1 custom_value
    SSH_OPTIONS_2 custom_value
    SSH_OPTIONS_3 custom_value

Host secondhost
    ANOTHER_OPTION custom_value

Host *host
    ANOTHER_OPTION custom_value

Host *
    CHANGE_DEFAULT custom_value
    # 可将用户名全放在这
    # User xinchen
```

## 3.使用
```bash
# 配合sshpass
sshpass -f $passfile ssh $firsthost
```