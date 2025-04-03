---
title: Linux(Armbian) mount SMB on boot
date: 2022-09-08 20:00:54
tags:
---

参考

> https://support.zadarastorage.com/hc/en-us/articles/213024986-How-to-Mount-a-SMB-Share-in-Ubuntu
> https://askubuntu.com/questions/1092278/mount-windows-share-as-guest-from-command-line
> https://askubuntu.com/questions/157128/proper-fstab-entry-to-mount-a-samba-share-on-boot

## install cifs-utils

```console
sudo apt-get install cifs-utils
```

## create mount point

```console
mkdir /media/nas001
```

## mount as guest

```console
sudo mount -t cifs -o rw,guest,vers=1.0 //192.168.1.1/usb_storage /media/nas001/
or
sudo mount -t cifs -o rw,guest,vers=2.0 //192.168.1.1/usb_storage /media/nas001/
```

NOTE: 用1.0, mount之后文件都是777,用2.0不是777
NOTE: umount by following command

```console
umount /media/nas001
```

## update /etc/fstab to auto mount on boot

add following line in /etc/fstab

```conf
//192.168.1.1/usb_storage /media/nas001/ cifs rw,guest,vers=2.0,uid=quxr,gid=quxr,noauto,x-systemd.automount 0 0
```

> https://bbs.archlinux.org/viewtopic.php?id=165700

```text
Instead of changing network management software, just use systemd's automount feature.  It will not attempt to mount the filesystem until it is accessed.  So assuming you don't try to go peeking into that mountpoint before the network is up, is should be fine.  Just add "noauto,x-systemd.automount" to the options in your fstab.  This functionality is very similar to autofs, but not quite as feature rich.
```

因为fstab的顺序在WIFI网络启动之前，所以无法自动mount网络共享的SMB文件。加上"noauto,x-systemd.automount"参数可以在启动的时候不自动mount，改为在访问的时候才mount。通常在WIFI启动前不会访问/media/nas001/

uid,gid后面可以跟username,groupname或者userid,groupid。mount之后权限变为指定user,group。不加的话默认是root。

test by following command

```console
sudo mount -a
```
