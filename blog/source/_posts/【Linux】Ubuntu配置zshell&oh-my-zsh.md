---
title: 【Linux】Ubuntu配置zshell&oh-my-zsh
tags: [Linux]
date: 2018-03-30
---

> zshell：https://archive.codeplex.com/?p=shell  
>
> oh-my-zsh: https://github.com/robbyrussell/oh-my-zsh
>
> 终极 Shell——Zsh: https://linuxtoy.org/archives/zsh.html
>
>Linux终极shell-Z Shell--用强大的zsh & oh-my-zsh把Bash换掉: https://blog.csdn.net/gatieme/article/details/52741221

## 查看当前系统的shells 

```bash
xinchen@ubuntu:~$ cat /etc/shells
# /etc/shells: valid login shells
/bin/sh
/bin/dash
/bin/bash
/bin/rbash
/usr/bin/tmux
/usr/bin/screen
```

##  安装zsh

```bash
sudo apt-get install zsh
```

## 切换bash为zsh
```bash
# usermod命令用于修改用户的基本信息
# 
# usermod -G staff newuser2  (将newuser2添加到组staff中)
# usermod -l newuser1 newuser (修改newuser的用户名为newuser1)
# usermod -L newuser1 (锁定账号newuser1)
# usermod -U newuser1 (解除对newuser1的锁定)
sudo usermod -s /bin/ash $USERNAME

# or
chsh -s /bin/zsh
chsh -s `which zsh`

# 切换回bash
chsh -s /bin/bash
```

## 安装oh-my-zsh

### 官网推荐
Oh My Zsh is installed by running one of the following commands in your terminal. You can install this via the command-line with either `curl` or `wget`.

```bash
# curl
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

# wget
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"

# The default location is ~/.oh-my-zsh (hidden in your home directory)
export ZSH="$HOME/.dotfiles/oh-my-zsh"; sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

### github下载

```bash
# git clone
git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh

# 备份已有的zshrc, 替换zshrc, 至此可直接使用
cp ~/.zshrc ~/.zshrc.orig
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc

# 直接使用脚本安装
cd oh-my-zsh/tools
./install.sh
```

## 主题

```bash
# Once you find a theme that you'd like to use, you will need to edit the ~/.zshrc file. You'll see an environment variable (all caps) in there that looks like:
ZSH_THEME="robbyrussell"

# If you're feeling feisty, you can let the computer select one randomly for you each time you open a new terminal window.
ZSH_THEME="random" # (...please let it be pie... please be some pie..)

# And if you want to pick random theme from a list of your favorite themes:
ZSH_THEME_RANDOM_CANDIDATES=(
  "robbyrussell"
  "agnoster"
)
```

## 插件
```bash
# ~/.zshrc
plugins=(git encode64 urltools)
```

## 升级
```bash
# By default, you will be prompted to check for upgrades every few weeks. If you would like oh-my-zsh to automatically upgrade itself without prompting you, set the following in your ~/.zshrc:

DISABLE_UPDATE_PROMPT=true

# To disable automatic upgrades, set the following in your ~/.zshrc:

DISABLE_AUTO_UPDATE=true

# If you'd like to upgrade at any point in time (maybe someone just released a new plugin and you don't want to wait a week?) you just need to run:

upgrade_oh_my_zsh
```
## 卸载
If you want to uninstall `oh-my-zsh`, just run `uninstall_oh_my_zsh` from the command-line. It will remove itself and revert your previous `bash` or `zsh` configuration.