---
title: 【Linux】Ubuntu18 安装后的步骤
tags: [Linux]
date: 2018-09-12
---

# 记录Linux安装后需要的东西，不定期更新

## 1.服务器基本操作
### 更换国内源
> 阿里源: http://mirrors.aliyun.com/ubuntu  
> 清华源: https://mirrors.tuna.tsinghua.edu.cn/ubuntu/  
> 网易源: http://mirrors.163.com/ubuntu/

```bash
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```


### openssh
```bash
sudo apt-get install openssh-server
```


### htop
```bash
sudo apt-get install htop
```

### zsh 终端

> [参考](https://melodyfff.github.io/2018/03/30/%E3%80%90Linux%E3%80%91Ubuntu%E9%85%8D%E7%BD%AEzshell&oh-my-zsh/)

### tmux 终端分屏
```bash
sudo apt-get install tmux
```

### Docker CE
> https://yq.aliyun.com/articles/110806

官方命令安装
```bash
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

手动帮助安装
#### ubuntu
```bash
# step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
# step 2: 安装GPG证书
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# Step 3: 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# Step 4: 更新并安装 Docker-CE
sudo apt-get -y update
sudo apt-get -y install docker-ce

# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# apt-cache madison docker-ce
#   docker-ce | 17.03.1~ce-0~ubuntu-bionic | http://mirrors.aliyun.com/docker-ce/linux/ubuntu bionic/stable amd64 Packages
#   docker-ce | 17.03.0~ce-0~ubuntu-bionic | http://mirrors.aliyun.com/docker-ce/linux/ubuntu bionic/stable amd64 Packages
# Step 2: 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.1~ce-0~ubuntu-bionic)
# sudo apt-get -y install docker-ce=[VERSION]
```
#### CentOS 7 (使用yum进行安装)
```bash
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装 Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce
# Step 4: 开启Docker服务
sudo service docker start

# 注意：
# 官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，你可以通过以下方式开启。同理可以开启各种测试版本等。
# vim /etc/yum.repos.d/docker-ce.repo
#   将 [docker-ce-test] 下方的 enabled=0 修改为 enabled=1
#
# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# yum list docker-ce.x86_64 --showduplicates | sort -r
#   Loading mirror speeds from cached hostfile
#   Loaded plugins: branch, fastestmirror, langpacks
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
#   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
#   Available Packages
# Step2 : 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.0.ce.1-1.el7.centos)
# sudo yum -y install docker-ce-[VERSION]
```

## 2.桌面应用

### 主题
> gnome: https://www.gnome-look.org  
> Flat Remix DARK GNOME theme: https://www.gnome-look.org/p/1197717/  
> 参考博客1: https://www.cnblogs.com/feipeng8848/p/8970556.html  
> 参考博客2: https://blog.csdn.net/zyqblog/article/details/80152016


```bash
sudo apt-get install gnome-tweak-tool

# need reboot
sudo apt-get install gnome-shell-extensions
```

### Chrome
```bash
sudo wget http://www.linuxidc.com/files/repo/google-chrome.list -P /etc/apt/sources.list.d/
wget -q -O - https://dl.google.com/linux/linux_signing_key.pub  | sudo apt-key add -
sudo apt update
sudo apt install google-chrome-stable
```

### deepin-wine(QQ、TIM、微信)
> github地址: https://github.com/wszqkzqk/deepin-wine-ubuntu  
> gitee地址: https://gitee.com/wszqkzqk/deepin-wine-for-ubuntu

### Redis Desktop Manager

```bash
wget https://github.com/uglide/RedisDesktopManager/releases/download/0.8.3/redis-desktop-manager_0.8.3-120_amd64.deb --no-check-certificate

# 出现缺少依赖,需要添加其他版本的源
sudo dpkg -i redis-desktop-manager_0.8.3-120_amd64.deb

sudo apt-get -f install
```

### Fiddler

> http://fiddler.wikidot.com/mono

```bash
# 安装mono
sudo apt-get install mono-complete
# 获取fiddler
wget http://ericlawrence.com/dl/MonoFiddler-v4484.zip
# 解压
unzip MonoFiddler-v4484.zip -d Fiddler & cd ./Fiddler
# 运行
mono Fiddler.exe
```

### WPS
> http://www.wps.cn/product/wpslinux/

## 参考链接
> [ubuntu中文论坛](http://forum.ubuntu.org.cn/)