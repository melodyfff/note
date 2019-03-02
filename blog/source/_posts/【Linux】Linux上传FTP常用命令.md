---
title: 【Linux】Linux上传FTP常用命令
tags: [Linux]
date: 2019-3-02
---

# Linux上传FTP常用命令

## curl
```bash
# 列出ftp服务器上的列表
curl ftp://$FTP_URL --u name:passwd
curl ftp://name:passwd@$FTP_url 

# 上传
curl --ftp-pasv -T filename.suffix ftp://$FTP_URL
curl [-u name:passwd] -T filename.suffix ftp://$FTP_URL

# 下载
# -o：将文件保存为命令行中指定的文件名的文件中
curl ftp://$FTP_URL/file -o file.zip

# 下载连续的文件
# -O：使用URL中默认的文件名保存文件到本地
curl ftp://$FTP_URL/path/[1-10].git -O
curl ftp://$FTP_URL/path/[1,2,3].git -O


# 删除
curl [-u name:passwd] ftp://$FTP_URL -X 'DELE filename.suffix'
```

## ftp
```bash
# 连接
ftp [host]
# anonymous(匿名用户)

# 上传
ftp> put local-file [remote-file] 

# 上传一批文件
ftp> mput local-files 

# 下载
ftp> get [remote-file] [local-file] 

# 下载一批文件
ftp> mget [remote-files] 

# 结束链接
ftp> close
ftp> bye
```

# lftp
```bash
# 登录

lftp ftp://user:password@site:port 
lftp user:password@site:port 
lftp site -p port -u user,password 
lftp site:port -u user,password 

#上传
lftp -c "put filename.suffix -o $FTP_URL"
#删除
lftp -c "rm $FTP_URL/filename.suffix"

```