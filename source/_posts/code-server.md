---
title: 在我的安卓手机上安装Code(code-server)
date: 2025-04-05 10:36:27
tags:
---

在我的安卓手机上安装Code(code-server)
## 安装termux

## 设置sshd

打开termux,安装sshd

```console
pkg install openssh
```

运行sshd,sshd默认端口8022

```console
sshd
```

获得手机ip,用户名，设置密码

```console
ifconfig
whoami
passwd
```

从电脑上登陆手机，注意不要让手机锁屏

```console
ssh <uid>@<host> -p8022
```

## 安装code-server

https://coder.com/docs/code-server/termux

```console
pkg install tur-repo
pkg install code-server
```

## 运行code-server

```console
code-server --bind-addr 0.0.0.0:8080
```

不加参数直接运行code-server之绑定127.0.0.1:8080

## 浏览器访问http://<手机IP>:8080

页面提示

```text
Welcome to code-server
Please log in below. Check the config file at /data/data/com.termux/files/home/.config/code-server/config.yaml for the password.
```

在config.yaml文件的最后可以找到密码
