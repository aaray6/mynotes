---
title: Android9 tvbox用termux,alist和kodi播放阿里云盘视频
date: 2023-02-12 05:44:56
tags:
---

Android9 tvbox用termux,alist和kodi播放阿里云盘视频

在Bilibili上看到LeoHao-o的教程“仅用安卓盒子/电视打造高清家庭影院”。根据其教程在魔百盒CM311-1sa ZG上设置成功。

## 下载安装

电视盒子是安卓9系统，根据Termux的说明，安卓11不能用Termux。

在F-Droid下载Termux和Termux:Boot

https://f-droid.org/packages/com.termux/
https://f-droid.org/packages/com.termux.boot/

在电视盒应用市场(当贝市场)下载Kodi并安装

通过U盘安装Termux和Termux:Boot。

## 设置Termux安装Alist

盒子打开应用Termux。第一次可能提示错误。再次打开安装boot-strap之后就能启动。

根据bilibili教程

```console
termux-change-repo
```

修改源。我选择BFSU源。

```console
apt update
apt upgrade
```

遇到提问直接回车选默认值

```console
pkg install alist
pkg install openssh
```

openssh是可选的，为了ssh远程连接。

运行sshd命令启动sshd
注: 监听端口是8022

远程登录电视盒子

> https://joeprevite.com/ssh-termux-from-computer/

ifconfig命令获取盒子IP
whoami命令获取username (u0_a34 on cm311-1sa, u0_a23 on e900v22c)
passwd命令设置密码

在电脑上用ssh <username>@IP -p8022登录

```console
# cm311-1sa
ssh u0_a33@192.168.1.42 -p8022
# e900v22c
ssh u0_a23@192.168.1.67 -p8022
```

### Termux中安装alist

```console
pkg install alist
```

## 设置alist和sshd自动开机启动

> https://wiki.termux.com/wiki/Termux:Boot

盒子里运行一次Termux:Boot并关闭

创建脚本~/.termux/boot/start-sshd内容如下

```shell
#!/data/data/com.termux/files/usr/bin/sh
termux-wake-lock
sshd
```

创建脚本~/.termux/boot/start-alist内容如下

```shell
#!/data/data/com.termux/files/usr/bin/sh
termux-wake-lock
alist server > ~/alist.log 2>&1 &
```

重启盒子就可以自动启动alist
看/data/data/com.termux/files/home/alist.log文件可以获取alist管理员密码。管理员帐号admin

或者用命令
alist admin显示密码
一定要在~下执行，如果在~/.termux/boot下执行，会在目录下生成data目录
这个额外的data目录会导致termux_boot脚本无法自动运行。删除~/.termux/boot目录下额外的data目录即可。

电脑上用浏览器打开 http://<盒子IP>:5244 ，输入admin和管理员密码进入alist管理页面。

修改默认密码

添加阿里云盘

## 设置KODI

打开电视盒子的Kodi应用。添加视频源。

源类型webdav (不加s)
IP: 127.0.0.1
端口: 5244
路径: dav
用户名: admin
密码: 管理员密码
