---
title: Ubuntu搭建匿名socket5代理
layout: post
categories: Linux
tags: 计算机网络
---
* TOC
{:toc}
前面使用nginx和squid配置的正向代理都是基于HTTP或者HTTPS协议，对于其他协议如UDP协议就无能为力了，这里介绍全能代理socket5，实现全面的白嫖！
<!-- more --> 

### 1.安装 

`sudo apt install dante-server`

### 2.创建日志文件夹

`sudo mkdir /var/log/sockd`

### 3.修改配置文件 

`sudo vi /etc/danted.conf`

```
logoutput: /var/log/sockd/sockd.log
internal: 服务器ip port = 1080
external: 服务器ip
method: username none
user.privileged: root # 一般为服务器用户名
user.notprivileged: nobody
user.libwrap: nobody
client pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    log: connect disconnect
}
pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0 port gt 1023
    command: bind
    log: connect disconnect
}
pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    command: connect udpassociate
    log: connect disconnect
}
block {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    log: connect error
}
```

### 4.启动

`sudo service danted start`

### 5.检测是否启动成功 

`netstat -anp | grep 1080`

另外通过执行命令`sudo tail -100f /var/log/sockd/sockd.log ` 可以查看详细连接日志