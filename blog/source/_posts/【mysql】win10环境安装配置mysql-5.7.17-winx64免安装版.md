---
title: 【mysql】win10环境安装配置mysql-5.7.17-winx64免安装版
tags: [mysql]
date: 2017-5-14
---

## mysql-5.7.17-winx64免安装版，win10环境下安装配置

- my.ini 配置文件放在bin文件同级目录下

```html
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8


[mysqld]

#安装目录
basedir = D:\dev\mysql
#数据存放目录  data目录是要单独创建的，记得是个空文件夹
datadir =D:\dev\mysql\data
#端口
port = 3306

# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 最大连接数量
max_connections = 100
#单个内存表的最大值限定
max_heap_table_size = 64M
#为每个线程分配的排序缓冲大小
sort_buffer_size = 8M
#join 连表操作的缓冲大小,根据实际业务来设置，默认8M
join_buffer_size = 32M
# sql查询缓存，如果提交的查询与几次中的某查询相同，并且在query缓存中存在，则直接返回缓存中的结果
query_cache_size = 64M
```
- 以管理员身份打开cmd窗口后，将目录切换到你解压文件的bin目录
- 执行 `mysqld -install` 
- 继续执行 `mysqld --initialize-insecure --user=mysql;` 
这一步在data目录中生产一些文件

#### 启动mysql
-  执行  `net start mysql` 启动服务
- 默认的root密码是空的。
- 执行 `mysql -uroot -p`

#### 修改root密码
- 切换到数据库`mysql`
- `use mysql`

##### 5.6之前的版本
`update user set password=password('123') where user='root';`

##### 5.7之后的版本
查看user表的字段 `desc user;`  
使用 `set password = password('admin');` 来设置密码。  
并刷新权限 `flush privileges;`
