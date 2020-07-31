---
title: anbox
date: 2020-07-30 14:29:05
tags:
---

![anbox](/myimages/anbox_1.png "anbox")

Anbox是Linux下的安桌“模拟器”。与其他模拟器不同，它不是虚拟机。性能较快。

[Anbox主页](https://anbox.io/)

## 声音

如果没有声音，可以参考[这个帖子](https://github.com/anbox/anbox/issues/904)

基本思路是，需要拷几个文件到anbox下，但是anbox文件系统是只读的，所以anbox提供了“Android rootfs overlay”，可以在Linux下添加文件到anbox文件系统中。

[Android rootfs overlay](https://docs.anbox.io/userguide/advanced/rootfs_overlay.html)

需要添加的文件在[这里 https://github.com/anbox/anbox/tree/master/android/media](https://github.com/anbox/anbox/tree/master/android/media)

## 支持ARM程序和安装Google Play

Anbox默认只能运行x86的程序，这个[教程 Anbox: How To Install Google Play Store And Enable ARM (libhoudini) Support, The Easy Way](https://www.linuxuprising.com/2018/07/anbox-how-to-install-google-play-store.html)提供了一个脚本，可以安装Google Play和libhoudini, libhoudini可以让anbox运行ARM的代码。

## 国内访问Google Play

首先需要科学上网，但是目前主流的几个科学上网都是支持Socks5协议，而anbox不支持socks5,所以需要先用privoxy把socks5转成http代理

### privoxy安装与配置

参考[使用 privoxy 转发 socks 到 http](http://einverne.github.io/post/2018/03/privoxy-forward-socks-to-http.html)

ubuntu下用sudo apt install privoxy命令安装

然后修改配置文件/etc/privoxy/config

增加一行
forward-socks5t / 127.0.0.1:2080 .

2080是sock5代理监听端口

修改一行
listen-address 127.0.0.1:8118
为
listen-address 0.0.0.0:8118

8118是privoxy监听端口

用以下命令重启privoxy
sudo /etc/init.d/privoxy restart

最重要的一步，如果ubuntu下的防火墙是开的，一定要增加一条规则

8118 允许 进入 任何地方

注:这一步设置完后，家里的其他设备比如iphone都可以设置192.168.1.6:8118为http代理上网

### 设置anbox代理

参考[It is not possible to set proxy settings for network connection](https://github.com/anbox/anbox/issues/398)

192.168.1.6是我的ubuntu IP地址

add the proxy with the following command:

adb shell settings put global http_proxy 192.168.1.6:8118

To remove it I used these commands:

adb shell settings delete global http_proxy
adb shell settings delete global global_http_proxy_host
adb shell settings delete global global_http_proxy_port

注:以下命令可以重启anbox
sudo systemctl restart snap.anbox.container-manager.service

都设完后，就可以打开anbox中的Google Play登录了

### 命令行使用privoxy

export http_proxy=http://127.0.0.1:8118 && curl ip.gs
