---
title: oracle云服务器挂载修复boot volumn
date: 2023-03-21 15:42:44
tags:
---

Oracle云服务器无法启动

原因是内存不足

## 解决无法启动

解决方法参考这个视频

> https://www.youtube.com/watch?v=NAWAsx3cKxE

先把boot-volumn从instance上移除，然后挂在另一个可以启动的instance上的block-volumn。

Web控制台会提示运行几个命令以便在Linux里访问新volumn.

在Linux里运行

```console
sudo iscsiadm -m node -T iqn.2015-02.oracle.boot:uefi -p 169.254.2.3:3260 -u
sudo iscsiadm -m node -o delete -T iqn.2015-02.oracle.boot:uefi -p 169.254.2.3:3260
Logging out of session [sid: 2, target: iqn.2015-02.oracle.boot:uefi, portal: 169.254.2.3,3260]
Logout of [sid: 2, target: iqn.2015-02.oracle.boot:uefi, portal: 169.254.2.3,3260] successful.
sudo iscsiadm -m node -o new -T iqn.2015-02.oracle.boot:uefi -p 169.254.2.4:3260
sudo iscsiadm -m node -o update -T iqn.2015-02.oracle.boot:uefi -n node.startup -v automatic
sudo iscsiadm -m node -T iqn.2015-02.oracle.boot:uefi -p 169.254.2.4:3260 -l
New iSCSI node [tcp:[hw=,ip=,net_if=,iscsi_if=default] 169.254.2.4,3260,-1 iqn.2015-02.oracle.boot:uefi] added
Logging in to [iface: default, target: iqn.2015-02.oracle.boot:uefi, portal: 169.254.2.4,3260]
Login to [iface: default, target: iqn.2015-02.oracle.boot:uefi, portal: 169.254.2.4,3260] successful.

mount /dev/sdb1 /mnt
```

如果不用了，在deattach之前，需要执行

```console
umount /mnt
sudo iscsiadm -m node -T iqn.2015-02.oracle.boot:uefi -p 169.254.2.4:3260 -u
sudo iscsiadm -m node -o delete -T iqn.2015-02.oracle.boot:uefi -p 169.254.2.4:3260
Logging out of session [sid: 3, target: iqn.2015-02.oracle.boot:uefi, portal: 169.254.2.4,3260]
Logout of [sid: 3, target: iqn.2015-02.oracle.boot:uefi, portal: 169.254.2.4,3260] successful.
```

为了减少启动的进程，把服务禁止

/etc/systemd/system/multi-user.target.wants目录下是链接。只要删除链接，就可以禁止服务自动启动

在新服务器下，目录是/mnt/etc/systemd/system/multi-user.target.wants

删除以下链接就可以不启动docker,节省很多内存。
containerd.service
docker.service 

接下来把block-volumn移除，重新回到起不来的instance下attach boot-volumn。然后启动

## 增加交换文件

> https://linuxize.com/post/create-a-linux-swap-file/

原来已经创建了1个G的/swapfile文件，但是没设置/etc/fstab

在/etc/fstab文件中增加一行就可以在启动时自动挂载swap文件

```/etc/fstab
/swapfile swap swap defaults 0 0
```

用以下命令临时挂载

```console
sudo swapon /swapfile
sudo swapon --show
NAME      TYPE  SIZE   USED PRIO
/swapfile file 1024M 477.7M   -2
```

## 恢复被禁用的服务

```console
sudo systemctl enable docker
sudo systemctl enable containerd
```
