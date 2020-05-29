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

TODO，因为安装没有意义，所以以后再说。

## 参考

> [群晖32位安装教程](https://vipiu.net/archives/2018/07/18/565.html)
