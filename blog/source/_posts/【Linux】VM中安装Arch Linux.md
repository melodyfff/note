---
title: 【Linux】VM中安装Arch Linux
tags: [Linux]
date: 2018-6-3
---

# Arch Linux

> 官网参考[archwiki](https://wiki.archlinux.org/index.php/Main_page_(简体中文))
> 参考https://blog.csdn.net/u011152627/article/details/18925121
> https://www.cnblogs.com/bluestorm/p/5929172.html
### VM ware 安装注意事项
版本一定要选择：
- 其他 Linux 3.x 内核
- 其他 Linux 3.x 内核
- 其他 Linux 3.x 内核

### 测试网络
```bash
# 测试是否联网
ping -c 4 www.baidu.com
# 找不到主机可尝试开启dhcp
systemctl enable dhcpcd.service
```

### 测试系统时间
```bash
# 检测系统时间
timedatectl status
# 用 systemd-timesyncd 确保系统时间是正确的
timedatectl set-ntp true
```

### 测试存储设备
```bash
lsblk
```

### 分区
```bash
# Linux cfdisk命令用于磁盘分区
cfdisk
# 选择doc 格式为mnt格式分区
# /dev/sda1   linux
# /dev/sda2   linux
# /dev/sda3   swp  交换分区


# 格式化分区
mkfs.ext4 /dev/sda1
mkfs.ext4 /dev/sda2

# 交换分区
mkswap /dev/sda3
swapon /dev/sda3

# 挂载分区
# 挂载根分区 /
mount /dev/sda1 /mnt

# 挂载home目录
mkdir /mnt/home
mount /dev/sda2 /mnt/home
```

### 修改软件镜像源
镜像源在`/etc/pacman.d/mirrorlist`

```bash
# 选出中国镜像
grep -A 1 'China' mirrorlist|grep -v '\-\-' >mirrorlisttemp
# 将源进行放到中国镜像末尾
cat mirrorlist>>mirrorlist2
mv mirrorlist2 mirrorlist
# 刷新pacman缓存
pacman -Syy
```

### 安装基本系统
```bash
# 联网下载,网速决定
# 最好确定第一行的源能连接，不然从第一个开始尝试连接卡顿很久
pacstrap -i /mnt base base-devel
```

### 配置系统
```bash
# 用以下命令生成 fstab 文件 (用 -U 或 -L 选项设置UUID 或卷标)
genfstab -U /mnt >> /mnt/etc/fstab
# 验证是否正确
nano /mnt/etc/fstab

# 进入新系统
arch-chroot /mnt /bin/bash

# 选择时区
tzselect
# 设置时区 上海
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
# 设置时间标准 为 UTC，并调整 时间漂移
hwclock --systohc --utc

# 设置Local
# 本地化的程序与库若要本地化文本，都依赖 Locale
# 后者明确规定地域、货币、时区日期的格式、字符排列方式和其他本地化标准等等。
# 在下面两个文件设置：locale.gen 与 locale.conf.
nano /etc/locale.gen
# en_US.UTF-8 UTF-8
# zh_CN.UTF-8 UTF-8
# zh_TW.UTF-8 UTF-8

# 生成locale信息
locale-gen

# 配置本地语言
echo LANG=en_US.UTF-8 > /etc/locale.conf


# 配置主机名
echo archlinux > /etc/hostname
# 将主机名添加进/etc/hosts
#<ip-address>   <hostname.domain.org>   <hostname>
# 127.0.0.1   localhost.localdomain   localhost archlinux
# ::1     localhost.localdomain   localhost archlinux

# 修改root密码
passwd

# 添加用户
useradd -m -g users -G wheel -s /bin/bash xinchen
# 安装sudo
pacman -S sudo 
# 将当前用户添加至/etc/sudoers 中 xinchen ALL=(ALL) ALL
nano /etc/sudoers

# 安装引导工具grub
pacman -S grub
# 注意这里的分区不需要指定分区数字
grub-install --recheck /dev/sda
# 生成默认配置文件
grub-mkconfig -o /boot/grub/grub.cfg

# 配置网络
systemctl enable dhcpcd.service

# 重启
# 输入 exit 或按 Ctrl+D 退出 chroot 环境。

# 可选用 umount -R /mnt 手动卸载被挂载的分区：这有助于发现任何“繁忙”的分区，并通过 fuser(1) 查找原因。

# 最后，通过执行 reboot 重启系统：systemd 将自动卸载仍然挂载的任何分区。不要忘记移除安装介质，然后使用root帐户登录到新系统。
```

### 安装桌面环境
所有的桌面环境都基于x11标准。x11是现在linux图形界面的标准，xorg是它的开源实现
|显卡|型号|
|:---:|:---:|
|AMD/ATI|xf86-video-amdgpu/xf86-video-ati|、
|Intel|xf86-video-intel|
|Nvidia|nvidia|

```bash
# 查询显卡型号
lspci | grep -e VGA -e 3D

# 虚拟机一般用pacman -S xf86-video-vesa就行
pacman -S xorg
pacman -S xorg-server xorg-xinit

# 字体搜索，选择要安装的字体
pacman -Ss fonts
```

GNOME桌面

```bash

# 可以不全部安装
pacman -S gnome gnome-extra
# gdm登录管理器
pacman -S gnome gdm
# 将gdm设置为开机自启动，这样开机时会自动载入桌面
# sysyemctl disable gdm 取消开机启动
systemctl enable gdm

# 图形界面终端Xterm
pacman -S xterm
```



