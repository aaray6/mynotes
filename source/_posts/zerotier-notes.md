---
title: zerotier笔记
date: 2023-03-21 15:39:14
tags:
---

官方网站地址：https://www.zerotier.com

注册帐号axxxy@2xxx.com/com8

每个免费套餐可以享受 100 台设备的内网互联，一般够用了。

** 注:我的主路由小米R3G(Padavan)始终无法让设备连上zerotier,但换成ZTE-E8820S(openwrt)之后就可以了。小米R3G换成openwrt系统后也可以正常连上zerotier**

## 网络配置

注册好之后，我们来建立一个 Network 并分配内网网段。

创建一个新的网络之后，我们会得到一个 Network ID。这是客户端连接到行星服务器的唯一识别码，需要牢记

## 客户端配置

ZeroTier 支持 Windows、macOS、Linux 三大桌面平台，iOS、Android 两大移动平台，QNAP（威连通）、Synology（群晖）、Western Digital MyCloud NAS（西部数据） 三个 NAS 平台，还支持 OpenWrt/LEDE 开源路由器项目。

下载地址：https://www.zerotier.com/download/

我在X201(ubuntu), N1(armbian), e900v22d(armbian)和P30(harmony os)上安装了客户端。

直接使用命令行进行操作的方法如下

```console
# 启动 (Linux下是用系统服务zerotier-one)
$ zerotier-one -d (不用此方法)
$ sudo systemctl start zerotier-one.service

# 获取地址和服务状态
$ zerotier-cli status

# 加入、离开、列出网络
$ zerotier-cli join # Network ID
$ zerotier-cli leave # Network ID
$ zerotier-cli listnetworks
```

## 认证设备和组网

回到一开始注册的网页，会发现设备列表当中新增了两台设备，在前面的方框打钩即可。根据 Node ID 判断设备的类型，可以修改设备被分配的 IP。

## Linux enable/disable zerotier

```console
# disable
sudo systemctl disable zerotier-one.service
# enable
sudo systemctl enable zerotier-one.service
# start
sudo systemctl start zerotier-one.service
# stop
sudo systemctl stop zerotier-one.service
# status
systemctl status zerotier-one.service
```
