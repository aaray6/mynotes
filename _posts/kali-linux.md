---
title: Kali linux
date: 2019-02-20 14:33:34
tags:
---

# Kali Linux Notes

Install Kali Linux on VirtualBox. The host is ubuntu

Set 1G memory and 2 CPUs.

## MT7601U Wireless Adapter in Kali in VirtualBox

MT7601U Wireless Adapter can be used in my host system. No need to do any setting. The ubuntu has driver already.

But in guest VirtualBox Kali linux, it can't work by default.

lsusb command shows the device after the adapter pluged and set in VirtualBox USB device.

```console
$ lsusb
Bus 001 Device 000: ID 148f:7601 Ralink Technology, Corp. MT7601U Wireless Adapter
Bus 001 Device 002: ID 80ee:0021 VirtualBox USB Tablet
Bus 001 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
```

but the iwconfig can't show the device.

```console
$ /sbin/iwconfig
lo        no wireless extensions.

eth0      no wireless extensions.

```

dmesg shows message like below

```console
$ dmesg|grep mt7
[ 188.388139] mt7601u: probe of 8-3:1.0 failed with error -110
```

The issue is not about the adapter driver. It's about the USB.

After change the VirtualBox USB device setting from USB2.0 (EHCI) Controller to USB1.1 (OHCI) Controller and restart guest system, it works.

![VirtualBox USB setting](/myimages/kali_linux_wireless_usb01.png)

```console
$ lsusb
Bus 001 Device 000: ID 148f:7601 Ralink Technology, Corp. MT7601U Wireless Adapter
Bus 001 Device 002: ID 80ee:0021 VirtualBox USB Tablet
Bus 001 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
$ /sbin/iwconfig
lo        no wireless extensions.

eth0      no wireless extensions.

wlan0     IEEE 802.11  ESSID:off/any  
          Mode:Managed  Access Point: Not-Associated   Tx-Power=20 dBm   
          Retry short limit:7   RTS thr:off   Fragment thr:off
          Power Management:off

```

## Install Chinese font

```console
sudo apt-get install ttf-wqy-zenhei
```

> [Kali之——kali精简版安装后中文乱码](https://blog.csdn.net/l1028386804/article/details/83107900)
> [kali linux系统中文乱码问题的解决](https://blog.csdn.net/qq_29343201/article/details/51880719)
