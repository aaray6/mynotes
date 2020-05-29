---
title: Linux/Windows虚拟机virtualbox上安装黑群晖synology
date: 2020-05-26 11:51:23
tags:
---

参考[用PC玩群晖NAS，虚拟机安装DSM](https://www.youtube.com/watch?v=7HeVI4J4r6M)
原文是用Windows下vmware安装。尝试用其提供的文件synoboot.img和DSM_DS3617xs_23739.pat并用VBoxManage转换img文件为vdi文件，用nmap命令取代群晖助手获得虚拟机mac地址。
 原文提供的下载,链接: https://pan.baidu.com/s/18xrtSiHxeIsP... 提取码: bhrg

## 下载启动文件和DSM文件并转换格式

```console
VBoxManage convertfromraw --format VDI synoboot.img synoboot.img.vdi
```

## 在virtualbox中创建虚拟机

创建虚拟机synology
类型Linux,版本Linux.2.6 / 3.x / 4.x (64-bit)
内存1024MB
使用已有的虚拟硬盘文件synoboot.img.vdi,这个是引导盘
点创建

然后设置刚创建的虚拟机synology
添加一个或多个虚拟硬盘用来存数据，删除光驱
禁用声音
修改网络，网卡1为桥接网卡，MAC地址需要之后修改（重要）

## 启动虚拟机

系统启动后，会停留在如下界面

```console
Screen will stop updateing shortly, please open http://find.synology.com to continue.
```

## 配置synology

### 方法1, http://find.synology.com

这个方法在成功安装DSM之前，总是找不到机器，页面总是显示局域网内未找到DiskStation

### 方法2,Windows下安装/运行synology-assistant-6.2-23733.exe

如果有Windows系统，用这个工具查找Synology服务器以及MAC地址

### 方法3,进入路由器界面查看

如果你有路由器权限，进入已连接设备页面查看，寻找设备名称是DISKSTATION的机器,记录MAC地址和IP地址

### 方法4,nmap

假如你没有Windows系统，没有路由器的访问权限，只有Linux可用，那可以用以下方法。

需要root权限

```console
xxxx@host:~/dev/blog$ sudo nmap -sP 192.168.1.1/26

Starting Nmap 7.60 ( https://nmap.org ) at 2020-05-26 17:09 CST
Nmap scan report for www.routerlogin.com (192.168.1.1)
Host is up (0.0032s latency).
MAC Address: A0:04:60:B6:E3:E0 (Netgear)
Nmap scan report for 192.168.1.5
Host is up (-0.066s latency).
MAC Address: 64:FB:81:61:B0:AF (Ieee Registration Authority)
Nmap scan report for 192.168.1.7
Host is up (0.12s latency).
MAC Address: 94:D0:29:C1:AC:AF (Guangdong Oppo Mobile Telecommunications)
Nmap scan report for 192.168.1.9
Host is up (0.13s latency).
MAC Address: 78:3A:84:C1:61:2B (Apple)
Nmap scan report for 192.168.1.11
Host is up (0.033s latency).
MAC Address: 44:59:E3:9D:1B:CE (Unknown)
Nmap scan report for 192.168.1.17
Host is up (-0.035s latency).
MAC Address: 40:31:3C:A1:45:A0 (Unknown)
Nmap scan report for 192.168.1.22
Host is up (0.00067s latency).
MAC Address: 58:94:6B:3D:79:DC (Intel Corporate)
Nmap scan report for 192.168.1.27
Host is up (0.10s latency).
MAC Address: 00:11:32:2C:A6:03 (Synology Incorporated)
Nmap scan report for 192.168.1.6
Host is up.
Nmap done: 64 IP addresses (9 hosts up) scanned in 29.88 seconds
```

192.168.1.27, MAC Address: 00:11:32:2C:A6:03 (Synology Incorporated)是我们要找的电脑

强制关闭virtualbox/synology虚拟机，修改synology虚拟机设置
网络-网卡1-高级-MAC 地址为0011322CA603
启动虚拟机

重新按照方法1, 浏览器打开http://find.synology.com就可以找到
![Synology Web Assistant](/myimages/synology_01.png "Synology Web Assistant")

点联机按钮，按照向导安装DSM_DS3617xs_23739.pat文件。
需要注意的是，中间选手动安装，然后选择DSM_DS3617xs_23739.pat文件。

会提示安装过程中，硬盘1 2中的所有数据将会被删除，是否确定继续？

打勾选确定。

大约10分钟安装，重启，系统就绪。

整个过程不截图了，可以看[参考](#参考)的第2个帖子，里面截图很详细。

重启后浏览器会自动刷新，进入创建管理员用户名密码阶段。

下一步选择“有DSM更新时通知我，让我手动安装”

下一步设置QuickConnect，选择跳过此步骤。

## 基本使用

第一步需要进入存储空间管理员

首先创建存储池，因为我只加了一个虚拟硬盘，所以只能选"Basic"作为RAID类型，根据向导，把加的那个虚拟硬盘拖到存储池中。

然后进入存储空间，在刚才创建的存储池中创建存储空间。

第二步创建共享文件夹

然后就可以在Windows的资源管理器中通过\\mynas2访问共享文件夹了。
在Ubuntu里可以通过smb://192.168.1.27访问共享文件夹。

## 其他应用

TODO

## 参考

> [用PC玩群晖NAS，虚拟机安装DSM](https://www.youtube.com/watch?v=7HeVI4J4r6M) 链接: https://pan.baidu.com/s/18xrtSiHxeIsP... 提取码: bhrg
>[VirtualBox安装黑群晖并建立smb共享目录的方法](https://blog.csdn.net/lvshaorong/article/details/82956312)
>备用DSM6.2软件下载地址 链接: [https://pan.baidu.com/s/1SOWHx9NvshEP1WrcT4lGJw](https://pan.baidu.com/s/1SOWHx9NvshEP1WrcT4lGJw)  密码: 4m6e
