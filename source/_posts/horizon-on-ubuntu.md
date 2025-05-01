---
title: Vmware(Omnissa) Horizon Client on ubuntu 24.04
date: 2025-05-01 14:40:12
tags:
---

由于Debian12下我的wifi经常出问题，所以换回ubuntu24.04

安装Horizon Client后遇到几个问题

## 不支持您正在使用的显示服务协议，建议配置X11显示法服务协议后登录

https://blog.csdn.net/Tears__Liu/article/details/129406330

编辑/etc/gdm3/custom.conf

把WaylandEnable=false前面的注释去掉，重启。

## VDPCONNECT_FAILURE

日志显示i965_drv_video.so找不到

https://forums.linuxmint.com/viewtopic.php?t=298620

```console
apt install i965-va-driver libvdpau-va-gl1
```

安装驱动后解决