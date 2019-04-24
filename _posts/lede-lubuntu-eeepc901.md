---
title: lede_lubuntu_eeepc901
date: 2019-04-23 17:53:04
tags:
---
# LEDE & Lubuntu on eeepc901

install LEDE and Lubuntu dual-boot on eeepc901

## 1. install lubuntu on /dev/sdb

## 2. 假设/dev/sda是空的，安装LEDE到/dev/sda上

### download LEDE x86

https://www.right.com.cn/forum/thread-252795-1-1.html
https://pan.baidu.com/s/1gfmuJOB 提取码:f33q

得到文件LEDE-17.01.2-R7.3.3-x86-combined-squashfs.rar
解压得到LEDE-17.01.2-R7.3.3-x86-combined-squashfs.img

### 假设lubuntu的grub是安装在/dev/sda上的，先备份mgr

用命令

```console
sudo dd bs=512 count=1 if=/dev/sda of=~/mbr_sda.bak
```

### 安装LEDE

```console
sudo dd bs=512 if=LEDE-17.01.2-R7.3.3-x86-combined-squashfs.img of=/dev/sda && sync
```

### 恢复mbr

```console
sudo dd bs=446 count=1 if=~/mbr_sda.bak of=/dev/sda && sync
```

## 3. 设置双启动

LEDE在/dev/sda上, /dev/sda1是ext4的boot分区，/dev/sda2是squashfs的/分区
Lubuntu在/dev/sdb上，/dev/sdb1是ext4的/分区

先进lubuntu，假如grub不在/dev/sda上，可以用以下命令安装grub,或者用以下命令确保grub安装到/dev/sda的mbr上

```console
# grub-install /dev/sda
```

增加LEDE菜单

检查LEDE原本的grub菜单配置，配置内容在/dev/sda1中，menuentry是我们要找的内容
用blkid获取新的/dev/sda2的PARTUUID
/dev/sda1/boot_rm/vmlinz是LEDE的引导内核，需要复制到lubuntu的/boot中
因为用lubuntu的update-grub无法自动获取并添加LEDE到grub菜单中，需要手工添加到/etc/grub.d/40_custom文件中
并且需要添加savedefault以便能够记录最后一次启动的选项
最后修改/etc/default/grub

```conf
GRUB_DEFAULT=saved
GRUB_SAVEDEFAULT=true
GRUB_TIMEOUT_STYLE=menu
```

并运行sudo update-grub

```console
# mount /dev/sda1 /mnt
# cat /mnt/boot_rm/grub/grub.cfg
serial --unit=0 --speed=115200 --word=8 --parity=no --stop=1 --rtscts=off
terminal_input console serial; terminal_output console serial

set default="0"
set timeout="0"
set root='(hd0,msdos1)'

menuentry "LEDE" {
	linux /boot/vmlinuz root=PARTUUID=5cd62fdc-02 rootfstype=squashfs rootwait console=tty0 console=ttyS0,115200n8 noinitrd
}
menuentry "LEDE (failsafe)" {
	linux /boot/vmlinuz failsafe=true root=PARTUUID=5cd62fdc-02 rootfstype=squashfs rootwait console=tty0 console=ttyS0,115200n8 noinitrd
}

# blkid
/dev/sda1: UUID="57f8f4bc-abf4-655f-bf67-946fc0f9f25b" TYPE="ext4" PARTUUID="ec399030-01"
/dev/sda2: TYPE="squashfs" PARTUUID="ec399030-02"
/dev/sda3: UUID="0314c9ce-bae9-42b8-bb4b-485f6fa23b3c" TYPE="swap" PARTUUID="ec399030-03"
/dev/sdb1: UUID="55cefc34-63db-44ac-af8e-20b3ab6472f5" TYPE="ext4" PARTUUID="f9d6ea38-01"

# cp /mnt/boot_rm/vmlinuz /boot

# vi /etc/grub.d/40_custom

# cat /etc/grub.d/40_custom

#!/bin/sh
exec tail -n +3 $0
# This file provides an easy way to add custom menu entries.  Simply type the
# menu entries you want to add after this comment.  Be careful not to change
# the 'exec tail' line above.

menuentry "LEDE" {
        savedefault
        linux /boot/vmlinuz root=PARTUUID=ec399030-02 rootfstype=squashfs rootwait console=tty0 console=ttyS0,115200n8 noinitrd
}
menuentry "LEDE (failsafe)" {
	linux /boot/vmlinuz failsafe=true root=PARTUUID=ec399030-02 rootfstype=squashfs rootwait console=tty0 console=ttyS0,115200n8 noinitrd
}

# vi /etc/default/grub

# more /etc/default/grub
GRUB_DEFAULT=saved
GRUB_SAVEDEFAULT=true
GRUB_TIMEOUT_STYLE=menu
GRUB_TIMEOUT=10
GRUB_DISTRIBUTOR='Lubuntu 18.10'
GRUB_CMDLINE_LINUX_DEFAULT="quiet resume=UUID=052fd2f3-ce2d-4b60-8af2-307
a64f6bd0a"
GRUB_CMDLINE_LINUX=""

# update-grub

# reboot
```

## 4. 设置LEDE

家里的主路由/默认网关是192.168.1.1
**eeepc901必须用网线接主路由**，eeepc901的LEDE作为旁路由，并且打开科学上网功能，IP是192.168.1.252。DHCP是不需要开启的，以避免跟主路由的DHCP冲突
其他机器无论是电脑，PS4还是iPad/iphone，如果需要科学上网，可以修改网络地址从自动获取改为手工配置。
只要设置默认网关是192.168.1.252，DNS1 192.168.1.252 DNS2 8.8.8.8就可以
但是我有一台公司电脑，因为没有管理员权限，所以无法手工更改IP设置，只能用DHCP的默认选项。这时需要打开LEDE的DHCP。并且设置让DHCP自动指定网关和DNS为192.168.1.252。这样当公司电脑连接旁路由的WIFI时，会被旁路由的DHCP分配IP以及设置网关/DNS
最后，可以在LEDE的DHCP服务中设置只给指定的MAC地址分配IP并把公司电脑的MAC地址添加进去，这样就对主路由的DHCP尽可能减少干扰。

### 修改LEDE IP

启动到LEDE中

update /etc/config/network文件

默认是192.168.1.1,更新以下IP避免跟网络默认路由冲突,修改后重启
config interface 'lan'
    ...
    option ipaddr '192.168.1.252'
    ...

### Web UI配置

浏览器打开192.168.1.252,默认用户名密码是root/password.可以到 系统->管理权中修改

![LEDE](/myimages/lede_01.png)

![LEDE](/myimages/lede_02.png)

![LEDE](/myimages/lede_03.png)

![LEDE](/myimages/lede_04.png)

根据需要在这里打开或者关闭DHCP,默认不需要DHCP,只有当我上面提到的那种无法手工设置网络参数时才需要打开。一个网段里有两个DHCP会互相有影响的。
![LEDE](/myimages/lede_05.png)

DHCP选项6,192.168.1.252意思是DNS:192.168.1.252

> [BOOTP/DHCP options](http://www.networksorcery.com/enp/protocol/bootp/options.htm)

![LEDE](/myimages/lede_06.png)

WIFI设置
![LEDE](/myimages/lede_07.png)

![LEDE](/myimages/lede_08.png)

![LEDE](/myimages/lede_09.png)

这里设置静态地址分配，可以只给指定MAC地址的机器分配DHCP IP
![LEDE](/myimages/lede_10.png)

科学上网
![LEDE](/myimages/lede_11.png)

![LEDE](/myimages/lede_12.png)

## 关于旁路由

旁路由除了可以像上面那样用物理机器，还可以用virtualbox/vmware等虚拟机。但是注意，**host主机必须用网线连接网络，虚拟机bridge的网卡也必须是有线网卡**,我尝试过host连wifi，无论怎样都无法让虚拟机里的旁路由正常工作，但只要换上有线，就没问题。原因不清楚。

如果用虚拟机做旁路由，对于我的那台没有管理员权限的公司电脑，需要有一个DHCP服务器给它设置IP/网关。
一个方法就是找一个WIFI路由器，连接LAN口到主路由的LAN口。然后打开副路由的DHCP，设置相应参数，当电脑连接副路由的WIFI时，就会被副路由的DHCP设置网关/DNS为虚拟机的旁路由IP.