---
title: eeepc901上用docker运行openwrt
date: 2021-10-28 18:08:53
tags:
---

eeepc901上用docker运行openwrt

eeepc901是32位x86 CPU

## 安装debian 11 32bit

下载debian 11 32bit ISO。
Linux下用balenaEtcher制作启动U盘。
eeepc901启动时按ESC,选择U盘启动，安装系统。
不需要选择X Server,只需要安装基本包和SSH Server。

## 安装docker 32bit

> https://stackoverflow.com/questions/37989534/how-to-install-docker-on-32bit-machine-having-ubuntu-12-04

root帐号下执行

```console
apt install -y docker.io
```

测试docker是否正常

```console
# docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (i386)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

## 安装docker下的应用

### portainer

常用的portainer无法安装在32位docker系统下

### 常用docker命令

docker --help
docker pull <image>
docker images ls
docker images rm <image>
docker container ls
docker container rm <container>
docker exec -it <container> ash

### openwrt

> https://github.com/SuLingGG/OpenWrt-Docker

#### 打开网卡混杂模式（这一步不是必须的）

```console
ip link set enp3s0 promisc on
```

#### 创建manvnet网络

```console
docker network create -d macvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1 -o parent=enp3s0 macnet
```

192.168.1.1是默认网管（路由器地址）。enp3s0是有线网卡名字，可以用ip addr查看所有网卡。**macvnet无法用无线网卡创建**。

#### 下载image

```console
docker pull sulinggg/openwrt:x86_generic
```

#### 启动openwrt

```console
docker run -d --name=openwrt --restart always --privileged --network macnet sulinggg/openwrt:x86_generic /sbin/init
```

#### 进入docker container修改ip

```console
docker exec -it openwrt ash
```

在openwrt container中

```console
# cat /etc/config/network

config interface 'loopback'
	option ifname 'lo'
	option proto 'static'
	option ipaddr '127.0.0.1'
	option netmask '255.0.0.0'

config globals 'globals'
	option packet_steering '1'

config interface 'lan'
	option type 'bridge'
	option ifname 'eth0'
	option proto 'static'
	option netmask '255.255.255.0'
	option ip6assign '60'
	option ipaddr '192.168.1.41'
	option gateway '192.168.1.1'
	option dns '192.168.1.1'

config interface 'vpn0'
	option ifname 'tun0'
	option proto 'none'


```

修改IP后重启网络

```console
/etc/init.d/network restart
```

#### 禁用DHCP

>> https://openwrt.org/docs/guide-user/network/wifi/dumbap

Disable DHCP Server
If you still need dnsmasq running for something else (e.g. TFTP server) you can do:

``` console
uci set dhcp.lan.ignore=1
uci commit dhcp
/etc/init.d/dnsmasq restart
```

我用的是这个方法。

If not disable dnsmasq service:

```console
/etc/init.d/dnsmasq disable
/etc/init.d/dnsmasq stop
```

这个方法尽管可以不启动dnsmasq服务。但是其他服务会用到dnsmasq,会在系统启动的时候自动把dnsmasq起来。

####  openwrt web console

http://192.168.1.41

默认用户名密码root/password

**在docker host上是无法通过macvnet访问guest的，所以需要在别的客户端访问openwrt**

### 让eeepc901合上盖子时不休眠

> https://blog.csdn.net/acxlm/article/details/78248819

1、在终端中编辑 logind.conf

```console
sudo vim /etc/systemd/logind.conf
```

2、打开文件后修改下面这行：

```config
#HandleLidSwitch=suspend
#修改后如下
HandleLidSwitch=ignore
```

3、保存后重启即可
