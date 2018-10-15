---
title: 【Linux】Nginx常用配置
tags: [Linux]
date: 2018-10-15
---

> nginx wiki: https://www.nginx.com/resources/wiki

## 1. 安装

> 官方指南: https://www.nginx.com/resources/wiki/start/topics/tutorials/install/

## 2. 官方完整示例
> https://www.nginx.com/resources/wiki/start/topics/examples/full/

#### nginx.conf
```bash
user  www www;
worker_processes  2;
pid /var/run/nginx.pid;

# [ debug | info | notice | warn | error | crit ]
error_log  /var/log/nginx.error_log  info;

events {
  worker_connections   2000;
  # use [ kqueue | rtsig | epoll | /dev/poll | select | poll ] ;
  use kqueue;
}

http {
  include       conf/mime.types;
  default_type  application/octet-stream;

  log_format main      '$remote_addr - $remote_user [$time_local]  '
    '"$request" $status $bytes_sent '
    '"$http_referer" "$http_user_agent" '
    '"$gzip_ratio"';

  log_format download  '$remote_addr - $remote_user [$time_local]  '
    '"$request" $status $bytes_sent '
    '"$http_referer" "$http_user_agent" '
    '"$http_range" "$sent_http_content_range"';

  client_header_timeout  3m;
  client_body_timeout    3m;
  send_timeout           3m;

  client_header_buffer_size    1k;
  large_client_header_buffers  4 4k;

  gzip on;
  gzip_min_length  1100;
  gzip_buffers     4 8k;
  gzip_types       text/plain;

  output_buffers   1 32k;
  postpone_output  1460;

  sendfile         on;
  tcp_nopush       on;

  tcp_nodelay      on;
  send_lowat       12000;

  keepalive_timeout  75 20;

  # lingering_time     30;
  # lingering_timeout  10;
  # reset_timedout_connection  on;


  server {
    listen        one.example.com;
    server_name   one.example.com  www.one.example.com;

    access_log   /var/log/nginx.access_log  main;

    location / {
      proxy_pass         http://127.0.0.1/;
      proxy_redirect     off;

      proxy_set_header   Host             $host;
      proxy_set_header   X-Real-IP        $remote_addr;
      # proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;

      client_max_body_size       10m;
      client_body_buffer_size    128k;

      client_body_temp_path      /var/nginx/client_body_temp;

      proxy_connect_timeout      90;
      proxy_send_timeout         90;
      proxy_read_timeout         90;
      proxy_send_lowat           12000;

      proxy_buffer_size          4k;
      proxy_buffers              4 32k;
      proxy_busy_buffers_size    64k;
      proxy_temp_file_write_size 64k;

      proxy_temp_path            /var/nginx/proxy_temp;

      charset  koi8-r;
    }

    error_page  404  /404.html;

    location /404.html {
      root  /spool/www;

      charset         on;
      source_charset  koi8-r;
    }

    location /old_stuff/ {
      rewrite   ^/old_stuff/(.*)$  /new_stuff/$1  permanent;
    }

    location /download/ {
      valid_referers  none  blocked  server_names  *.example.com;

      if ($invalid_referer) {
        #rewrite   ^/   http://www.example.com/;
        return   403;
      }

      # rewrite_log  on;
      # rewrite /download/*/mp3/*.any_ext to /download/*/mp3/*.mp3
      rewrite ^/(download/.*)/mp3/(.*)\..*$ /$1/mp3/$2.mp3 break;

      root         /spool/www;
      # autoindex    on;
      access_log   /var/log/nginx-download.access_log  download;
    }

    location ~* ^.+\.(jpg|jpeg|gif)$ {
      root         /spool/www;
      access_log   off;
      expires      30d;
    }
  }
}
```

## 3. 简单自用反向代理配置
```bash
events {
    worker_connections  1024;
}
http {

include       mime.types;  
default_type  application/octet-stream; 

upstream hello  {
    server 192.168.201.130; 
}
 
## Start xinchen.sso.com ##
server {
    listen 80;
    server_name  xinchen.sso.com;
 
    access_log  logs/xinchen.access.log;
    error_log  logs/xinchen.error.log;
    root   html;
    index  index.html index.htm index.php;
 
    ## send request back to apache ##
    location / {
        proxy_pass  http://192.168.201.130;
 
        #Proxy Settings
        proxy_redirect     off;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_max_temp_file_size 0;
        proxy_connect_timeout      90;
        proxy_send_timeout         90;
        proxy_read_timeout         90;
        proxy_buffer_size          4k;
        proxy_buffers              4 32k;
        proxy_busy_buffers_size    64k;
        proxy_temp_file_write_size 64k;
   }
}
}
```