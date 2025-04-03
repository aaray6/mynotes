---
title: 在ZTE-E8820S/小米R3G上的openwrt上mount NTFS移动硬盘并设置网络共享
date: 2023-03-21 15:44:59
tags:
---

在ZTE-E8820S/小米R3G上的openwrt上mount NTFS移动硬盘并设置网络共享

## openwrt

ZTE-E8820S用的是恩山论坛上提供的固件，这个固件可以安装软件包。

> https://www.right.com.cn/forum/thread-8178758-1-1.html

## 挂载NTFS移动硬盘

E8820S的USB是2.0的不是3.0。

> https://openwrt.org/docs/guide-user/storage/writable_ntfs

安装ntfs-3g和fdisk包。安装ttyd便于使用命令行操作。

编辑脚本/etc/hotplug.d/block/10-mount内容如下

```shell
#!/bin/sh
# Copyright (C) 2011 OpenWrt.org
sleep 10 #more apps installed, need more time to load kernel modules!
blkdev=`dirname $DEVPATH`
if [ `basename $blkdev` != "block" ]; then
	device=`basename $DEVPATH`
	case "$ACTION" in
		add)
			mkdir -p /mnt/$device
			# vfat & ntfs-3g check
			if [ `which fdisk` ]; then
				isntfs=`fdisk -l | grep $device | grep NTFS`
				isvfat=`fdisk -l | grep $device | grep FAT`
				isfuse=`lsmod | grep fuse`
				isntfs3g=`which ntfs-3g`
			else
				isntfs=""
				isvfat=""
			fi

			# mount with ntfs-3g if possible, else with default mount
			if [ "$isntfs" -a "$isfuse" -a "$isntfs3g" ]; then
				ntfs-3g /dev/$device /mnt/$device
			elif [ "$isvfat" ]; then
				mount -o iocharset=utf8 /dev/$device /mnt/$device
			else
				mount /dev/$device /mnt/$device
			fi
		;;
		remove)
			umount -l /dev/$device
		;;
	esac
fi
```

插拔硬盘之后会自动mount到/mnt/sda1目录下

```console
# ls -l /mnt/sda1
drwxrwxrwx    1 root     root          4096 Feb 20 08:46 nas
drwxrwxrwx    1 root     root          4096 Apr 10  2019 temp
# 
```

## 文件共享

### 创建帐户

> https://openwrt.org/docs/guide-user/additional-software/create-new-users

```console
# opkg update
# opkg install shadow-useradd
# 
 user
```

如果没安装samba,安装如下包

```console
opkg install samba4-server luci-app-samba4 luci-i18n-samba4-zh-cn
```

添加user账户到samba账户中

```console
smbpasswd -a user
```

根据提示设置密码

### 共享设置

服务->网络共享

修改接口为lan

![修改接口](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/03/14/openwrt_samba1-1678788341673.png)

点新增按钮，增加

名称: nas
路径: /mnt/sda1/nas
允许用户: user

点保存并应用

![增加共享](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/03/14/openwrt_samba2-1678788752058.png)

### ubuntu下访问

打开文件->其他位置

连接到服务器输入: smb://192.168.1.1

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/03/14/openwrt_samba3-1678788969286.png)

回车或者按连接按钮

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/03/14/openwrt_samba4-1678789009383.png)

点击nas，输入用户名密码即可访问

## 小米R3G

方法基本跟上面的相同。因为可以自己编译固件，所以在编译的时候可以把需要的软件编译进去。

添加automount，可以取代上面的hotplug脚本，功能类似。
最好不添加autosamba，因为autosamba会自动增加sda1的共享，而我需要手动添加名字是nas的共享。
添加了一个
添加kmod-usb-storage-uas, kmod-usb-xhci-hcd模块，为了支持usb3.0的硬盘
