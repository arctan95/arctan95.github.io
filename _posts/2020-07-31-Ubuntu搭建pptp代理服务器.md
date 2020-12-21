---
title: Ubuntu搭建pptp代理服务器
layout: post
categories: Linux
tags: 计算机网络
---
* TOC
{:toc}
最近使用Ubuntu搭建了pptp代理服务器，成功绕过了校园网认证（白嫖使我快乐），这里记录一下主要步骤：

#### 1. 安装pptp

```sudo apt-get install pptpd```

#### 2. 修改pptpd.conf中的配置信息

```sudo vim /etc/pptpd.conf``` 
<!-- more -->
在末尾增加下面两行，或者打开的内容里面找到这两行，取消掉注释

```
localip  192.168.0.1
remoteip 192.168.0.234-238,192.168.0.245
```

分别为创建pptp时的主机ip和连接pptp的其他主机使用的ip段，可以自行修改。
注意，这里的ip并不是指外网ip或者当前局域网ip，而是指创建（虚拟专用网络）会分配的ip地址。一般这个可以不用修改。

#### 3. 修改chap-secrets配置

连接pptp 所需要的账号和密码，修改配置文件/etc/ppp/chap-secrets

```sudo vim /etc/ppp/chap-secrets```

在末尾添加以下内容

```
#用户名 pptpd 密码 *
tan  pptpd  123456  *
```

末尾的`*`表示可以使用任意IP连入，如果你要设置指定IP才能连接，可以将替换成对应的IP。支持添加多个账号。

#### 4. 设置ms-dns

配置使用的dns，修改配置文件

```sudo vim /etc/ppp/pptpd-options```

在末尾增加下面两行，或者打开的内容里面找到这两行，取消掉注释

```
# 这是谷歌的DNS 可以根据实际填写
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```

#### 5. 开启转发

修改配置文件

```sudo vim /etc/sysctl.conf```

在末尾增加下面内容，或者打开的内容里面找到这一行，取消掉注释

```
net.ipv4.ip_forward=1
```

保存之后执行

```sudo sysctl -p```

#### 6. 配置iptables

若未安装iptables 执行脚本安装



```sudo apt-get install iptables```



> tips：若之前安装pptp失败的。执行以下脚本；如果是第一次安装可忽略以下内容（目的为了清除iptables里旧的规则）





```
 1. sudo iptables -F
 2. sudo iptables -X
 3. sudo iptables -t nat -F
 4. sudo iptables -t nat -X
```

然后，允许GRE协议以及1723端口、47端口：

```
sudo iptables -A INPUT -p gre -j ACCEPT 
sudo iptables -A INPUT -p tcp --dport 1723 -j ACCEPT 
sudo iptables -A INPUT -p tcp --dport 47 -j ACCEPT
```

下一步，开启NAT转发：

```
sudo iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eno33 -j MASQUERADE
```

注意，上面的eno33是连接网络的网卡的名称，不同机器这个可能是不一样的。可以在终端输入ifconfig来查看
> tips：iptables里的规则在重启后会失效，所以可把这些规则写入开机启动命令中，Ubuntu可以通过编辑`/etc/rc.local`来实现。 


#### 7. 重启pptp服务

```sudo service pptpd restart```

#### 8.无法上网问题解决
有时会出现VPN可以拨上，而且可以 ping 通外网，但上网速度会很慢，很多页面打不开的情况，这是因为MTU过大，将MTU的值配置为1356即可。可以在iptables里增加如下规则： 

```
sudo iptables -I FORWARD -p tcp --syn -i ppp+ -j TCPMSS --set-mss 1356
```

不过最终iptables中 mss 的设置还是要以服务端的默认配置为准，通过netstat -i 查看得到ppp0口的mtu值 ，再做mtu-20字节的IP头部-20字节的TCP头部=1356 ，计算出iptables中需要设置的mss的值。

#### 9.PPTP客户端连接VPN
windows或手机VPN客户端的配置较简单，这里主要记录下linux上VPN客户端的配置

首先需要安装客户端:

```sudo apt-get install pptp-linux```

然后创建配置文件:

```sudo vim /etc/ppp/peers/pptp```

进行如下配置
```
// 添加如下内容：（自行更改IP, name, password）
pty "pptp xxx.xxx.xxx.xxx --nolaunchpppd"
name xxx
password xxx
remotename pptp
require-mppe-128
require-mschap-v2
refuse-eap
refuse-pap
refuse-chap
refuse-mschap
noauth
persist
maxfail 0
defaultroute
replacedefaultroute
usepeerdns
```
也可以使用```pptpsetup --create pptp --server 你的服务器 --username 你的账户 --password 你的密码 --encrypt --start```创建连接，并且执行完之后在```/etc/ppp/chap-secrets```这个文件会多出两行，第一行是注释掉的说明，第二行是参数，格式如此：```你的账户 pptp "你的密码" *```，注意别掉了星号。还有在```/etc/ppp/peers/```这个目录下多出一个以连接名称命名的文件

启动 & 关闭

```
启动：sudo pon pptp
关闭：sudo poff pptp
route add -net 0.0.0.0 dev ppp0 //添加默认路由
```
