---
title: 【Linux】Ubuntu的recovery mode修改系统文件
tags: [Linux]
date: 2018-12-29
---

#### 近日,由于安装网易云音乐的时候，修改了 `/etc/sudoers`，保存时没注意到语法错误。导致`sudo`命令无法使用。
#### 此时可以进入`recovery mode`模式进行修改系统文件


> 参考：https://www.jianshu.com/p/66ac9441fd1b

包括不限于以下几种场景：
- 在当前用户下使用sudo来直接修改password等几个文件，一旦修改了passwd，用户名发生了变化，其他的用户组、密码等却没有对应的配置，就再进不了该用户了
- 忘记用户密码，不能进入ubuntu了
- Ubuntu下普通用户用sudo执行命令时报"xxx is not in the sudoers file.This incident will be reported"错误

下面以修改`/etc/sudoers`文件问题为例

## 1. 步骤
- ① 重启电脑
- ② 开机时按下`esc`，进入`Grub`引导界面，选择`Advanced option for ubuntu`
- ③ 选择 `...(revocery mode)`
- ④ 选择`root drop to a root shell prompt`
- ⑤ 输入`~#`(也可不输入)
- ⑥ `mount -o remount,rw /`(重新挂载/etc分区，/etc是在根目录下(ubuntu 用 / 表示，所以是对/目录重新挂载为读/写)
- ⑦ `chmod u+w /etc/sudoers`
- ⑧ `vim /etc/sudoers`编辑文件，修改错误
- ⑨ `chmod u-w /etc/sudoers`
- ⑩ `mount -o remount, ro /` (重新挂载为只读)


