---
title: Nginx配置正向代理支持HTTP和HTTPS转发
layout: post
categories: Linux
tags: 计算机网络
---
* TOC
{:toc} 

前段时间使用PPTP实现了绕过校园网认证上网，最近又实现了通过设置正向代理的方式实现绕过校园网认证！这里记录下使用nginx搭建正向代理的过程。

Nginx本身不支持HTTPS正向代理，需要安装`ngx_http_proxy_connect_module`模块后才可以支持HTTPS正向代理，否则会遇到`HTTP 400`错误。

参考文档：

- https://github.com/chobits/ngx_http_proxy_connect_module

<!-- more -->

## 安装编译环境和工具

```
yum install gcc gcc-c++ autoconf automake -y 

yum install pcre pcre-devel -y 

yum install openssl openssl-devel -y 

yum install patch -y yum install git -y 

yum install net-tools -y

```

## 安装Nginx和ngx_http_proxy_connect_module模块

```
mkdir -p /downloads
cd /downloads

git clone https://github.com/chobits/ngx_http_proxy_connect_module.git

wget http://nginx.org/download/nginx-1.15.12.tar.gz
tar -xzvf nginx-1.15.12.tar.gz
cd nginx-1.15.12/

patch -p1 < /downloads/ngx_http_proxy_connect_module/patch/proxy_connect_rewrite_101504.patch

./configure --add-module=/downloads/ngx_http_proxy_connect_module

make && make install
```

## 修改Nginx配置文件

Nginx目录：`/usr/local/nginx`

修改Nginx目录下`conf/nginx.conf`配置文件，在`http`中添加以下内容：

```
server {  
    resolver 114.114.114.114; 
    listen 8080; #端口号可更改 
    location / {  
        proxy_pass http://$http_host$request_uri;
        proxy_set_header HOST $http_host;
        proxy_buffers 256 4k;
        proxy_max_temp_file_size 0k; 
        proxy_connect_timeout 30;
        proxy_send_timeout 60;
        proxy_read_timeout 60;
        proxy_next_upstream error timeout invalid_header http_502;
    }  
}


server {
     listen 8443;

     # dns resolver used by forward proxying
     resolver 114.114.114.114;

     # forward proxy for CONNECT request
     proxy_connect;
     proxy_connect_allow            443 563;
     proxy_connect_connect_timeout  10s;
     proxy_connect_read_timeout     10s;
     proxy_connect_send_timeout     10s;

     # forward proxy for non-CONNECT request
     location / {
         proxy_pass http://$host;
         proxy_set_header Host $host;
     }
 }
```

## 启动nginx

运行`/usr/local/nginx/sbin/nginx`启动nginx。

Nginx命令参考：

```
# Start Nginx
/usr/local/nginx/sbin/nginx

# Reload Nginx configuration
/usr/local/nginx/sbin/nginx -s reload

# Stop Nginx
/usr/local/nginx/sbin/nginx -s stop
```

## 查看端口

```
netstat -tnlp | grep 8080
netstat -tnlp | grep 8443
```

## 打开防火墙

```
firewall-cmd --zone=public --add-port=8080/tcp
firewall-cmd --zone=public --add-port=8080/tcp --permanent

firewall-cmd --zone=public --add-port=8443/tcp
firewall-cmd --zone=public --add-port=8443/tcp --permanent


firewall-cmd --reload
```

## 测试代理

```
# Test HTTP proxy
curl --proxy 192.168.87.123:8080 http://www.baidu.com

# Test HTTPS proxy
curl --proxy 192.168.87.123:8443 https://www.baidu.com
```
