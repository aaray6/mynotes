---
title: synology 32位eeepc901做NAS尝试
date: 2020-05-28 16:09:24
tags:
---

还是我的那台eeepc901,尝试了虚拟机装黑群晖后，想把eeepc改造成黑群晖。但是最终没有尝试完，因为群晖不能使用外接usb硬盘，只能用内部存储，而我的eeepc901是4G+16G，就算变成NAS，也没有什么实用价值。
所以在尝试到虚拟机后，暂停了。

## 下载需要的引导和系统文件

因为eeepc901的CPU是N720，是32位的，所以必须找32位的系统才行。最新的群晖系统只支持64位的。我能找到的只有5.0的32位系统。

下载32位5.0黑群晖文件 [链接: https://pan.baidu.com/s/1sc-EC5upNVsHDQUyolmIVg](https://pan.baidu.com/s/1sc-EC5upNVsHDQUyolmIVg) 密码: 6ode

文件如下
NB_x86_5024_DSM_50-4528_Xpenology_nl.img
DSM_DS214play_4528.pat

第一个img是引导文件，第2个pat是系统。

## 新建virtualbox 32位虚拟机

虚拟机安装方法基本上与"Linux/Windows虚拟机virtualbox上安装黑群晖synology"类似

先用如下命令把img转为vdi文件
VBoxManage convertfromraw --format VDI NB_x86_5024_DSM_50-4528_Xpenology_nl.img DSM_50-4528_Xpenology_nl.img.vdi

然后创建虚拟机syno32，这次选择Linux2.6/3.x/4.x (32-bit),使用上面的vdi作为第一个硬盘，再加一个SATA硬盘作为数据盘。

启动虚拟机，浏览器打开http://find.synology.com，会找到NAS,根据向导安装pat文件即可。

安装好之后，用sudo nmap -sP 192.168.1.1/24命令找到NAS服务器IP，浏览器打开http://<ip>:5000进入系统

## eeepc901安装

因为安装了也没有意义，所以没真正安装。

用img文件制作启动U盘
找了个1G TF卡,插上后用命令
sudo fdisk -l
获得U盘是/dev/sdb

```console
sudo fdisk -l
...
Disk /dev/sdb：971.5 MiB，1018691584 字节，1989632 个扇区
单元：扇区 / 1 * 512 = 512 字节
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x00468ae6
```

用dd命令写入img

```console
$ sudo dd bs=512 if=NB_x86_5024_DSM_50-4528_Xpenology_nl.img of=/dev/sdb && sync
记录了65536+0 的读入
记录了65536+0 的写出
33554432 bytes (34 MB, 32 MiB) copied, 7.17033 s, 4.7 MB/s
```

把U盘和网线插到eeepc901上，开机按ESC，选择U盘启动。剩下的跟在虚拟机类似。

WIFI可能是不能驱动，所以必须插网线，启动后在屏幕上有IP地址。也可以用其他方法。

```console
$ sudo nmap -sP 192.168.1.1/26

Starting Nmap 7.60 ( https://nmap.org ) at 2020-05-30 14:51 CST
...
Nmap scan report for 192.168.1.14
Host is up (-0.066s latency).
MAC Address: 00:22:15:XX:XX:XX (Asustek Computer)
...
Nmap done: 64 IP addresses (6 hosts up) scanned in 31.87 seconds
```

或者浏览器打开 http://find.synology.com/ ， 可以找到eeepc901,然后根据向导安装。

后续步骤不试了。

## 参考

> [群晖32位安装教程](https://vipiu.net/archives/2018/07/18/565.html)
