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

## add new user by root and add sudo group

```console
# useradd newuser
# /sbin/usermod -a -G sudo newuser
# passwd newuser

```

## wordlist

> [SecLists](https://github.com/danielmiessler/SecLists)

### Install

** Zip **

```console
wget -c https://github.com/danielmiessler/SecLists/archive/master.zip -O SecList.zip \
  && unzip SecList.zip \
  && rm -f SecList.zip
```

** Git (Small) **

```console
git clone --depth 1 https://github.com/danielmiessler/SecLists.git
```

** Git (Complete) **

```console
git clone git@github.com:danielmiessler/SecLists.git
```

** Kali Linux **
([Tool Page](https://tools.kali.org/password-attacks/seclists))

```console
apt -y install seclists
```

## Kali Linux install VirtualBox addition fails

Both VirtualBox 5.2.x and 6.0.x addition show error message

```console
VirtualBox Guest Additions: modprobe vboxsf failed
```

The /var/log/vboxadd-setup.log shows

```console
/tmp/vbox.0/utils.c: In function ‘sf_init_inode’:
/tmp/vbox.0/utils.c:165:28: error: passing argument 1 of ‘sf_ftime_from_timespec’ from incompatible pointer type [-Werror=incompatible-pointer-types]
     sf_ftime_from_timespec(&inode->i_atime, &info->AccessTime);
                            ^~~~~~~~~~~~~~~
/tmp/vbox.0/utils.c:53:53: note: expected ‘struct timespec *’ but argument is of type ‘struct timespec64 *’
 static void sf_ftime_from_timespec(struct timespec *tv, RTTIMESPEC *ts)

```

Reason is the new Kali linux kernal version is 4.19.x, the virtualbox addtion utils.c code has issue from kernal version 4.18

** solution **

Download the Development version addition iso (5.2.97 and 6.0.29). Use this Addition iso. The issue is fixed in latest development version.

[Virtualbox download page](https://www.virtualbox.org/wiki/Testbuilds#Developmentsnapshots)

[VBoxGuestAdditions_6.0.97-128917.iso for VirtualBox6.0.x](https://www.virtualbox.org/download/testcase/VBoxGuestAdditions_6.0.97-128917.iso). So far (2019.02.21), I install latest VirtualBox6.0.4, the Addition in the package doesn't work. 6.0.97 Addition works.

> [topic](https://forums.virtualbox.org/viewtopic.php?f=3&t=89455)
> [topic, Simon South's answer](https://superuser.com/questions/1268741/virtualbox-ubuntu-guest-additions-not-installing-modprobe-vboxsf-failed)
> [ticket](https://www.virtualbox.org/ticket/17981)

## How to upgrade VirtualBox from 5.2 to 6.0

> [VirtualBox 6.0 Is Out — Here’s How To Install / Upgrade On Ubuntu 16.04 / 18.04 / 18.10](https://websiteforstudents.com/virtualbox-6-0-is-out-heres-how-to-install-upgrade-on-ubuntu-16-04-18-04-18-10/)

Use following command to get current Virtualbox version

```console
$ vboxmanage --version
6.0.4r128413
```