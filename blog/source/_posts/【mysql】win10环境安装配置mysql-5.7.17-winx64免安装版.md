---
title: 【mysql】win10环境安装配置mysql-5.7.17-winx64免安装版
tags: [mysql]
date: 2017-5-14
---
## 用以下命令检查是否安装
```html
sudo netstat -tap | grep mysql
```
## 安装命令
```html
sudo apt-get install MySQL-server mysql-client
```
## 查看安装端口情况
```html
sudo netstat -tap | grep mysql
```
## 进入mysql
```html
mysql -u root -p
```
# 配置远程访问
##  编辑mysql配置文件，把其中bind-address = 127.0.0.1注释了
```html
vi /etc/mysql/mysql.conf.d/mysqld.cnf 
```
## 进入MySQL输入以下
```html
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```
## 重启mysql
```html
sudo /etc/init.d/mysql restart
```

# 打开关闭服务
```html
sudo /etc/init.d/mysql start|stop
```

# 配置文件位置
```html
sudo vim /etc/mysql/my.cnf
```

# 其他文件默认位置
```html
/usr/bin                 客户端程序和脚本  
/usr/sbin                mysqld 服务器  
/var/lib/mysql           日志文件，数据库  ［重点要知道这个］  
/usr/share/doc/packages  文档  
/usr/include/mysql       包含( 头) 文件  
/usr/lib/mysql           库  
/usr/share/mysql         错误消息和字符集文件  
/usr/share/sql-bench     基准程序  
```

# 卸载
```html
sudo apt-get autoremove --purge mysql-server-5.5.43  
sudo apt-get remove mysql-server  
sudo apt-get autoremovemysql-server  
sudo apt-get remove mysql-common  
dpkg -l | grep ^rc| awk '{print $2}' | sudoxargsdpkg -P  
```