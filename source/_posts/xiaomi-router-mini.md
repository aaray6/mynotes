---
title: 小米路由器mini刷breed和openwrt
date: 2023-02-23 09:33:58
tags:
---

小米路由器mini刷breed和openwrt

> https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=690605
> https://blog.csdn.net/weixin_53742409/article/details/121676620
> https://zhuanlan.zhihu.com/p/237513064

## 小米升级开发版固件

在http://www1.miwifi.com/miwifi_download.html 网页找到小米路由器mini 开发版固件并下载
miwifi_r1cm_firmware_2e9b9_2.21.109.bin

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/22/xiaomi_mini_2_devrom-1677029325096.png)

在 Web 路由器管理页面的 常用设置—系统状态—升级检测—手动升级—上传安装固件—升级 ，等待安装完重启就行。如果无法进入系统，可以根据官方教程用 U 盘刷机。

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/22/xiaomi_mini_2_devrom_webupgrade-1677029363531.png)

选择刚下载的开发版rom升级,不选择擦除旧配置,提示需要花费几分钟。

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/22/xiaomi_mini_1_sshtool-1677029523944.png)

升级之后重新登录web控制台，显示版本号已经更新了。

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/22/xiaomi_mini_2_devrom_webupgrade_2-1677033255342.png)

## 小米官方ssh工具

### 绑定路由器
注册小米账号
去 Miwifi.com 下载手机客户端并登陆
绑定路由器
打开 http://d.miwifi.com/rom/ssh 并登陆获取路由器的 root 密码及工具包

注: 如果chrome浏览器无法下载，换firefox浏览器。

小米ID：86nnn39
已绑定1台小米路由器
RAY3(小米路由器mini)root密码 2cxxxxx8

### 工具包使用方法

工具包使用方法：小米路由器需升级到开发版0.5.28及以上，小米路由器mini需升级到开发版0.3.84及以上，小米路由器3即将支持。注意：稳定版不支持。
请将下载的工具包bin文件复制到U盘（FAT/FAT32格式）的根目录下，保证文件名为miwifi_ssh.bin；
断开小米路由器的电源，将U盘插入USB接口；
按住reset按钮之后重新接入电源，指示灯变为黄色闪烁状态即可松开reset键；
等待3-5秒后安装完成之后，小米路由器会自动重启，之后您就可以尽情折腾啦 ：）

## 连接路由器

用具有 SSH 功能的软件连接路由器，小米路由器的默认 IP段 都是 192.168.31.x

但因为我之前升级开发版固件时保留了设置，所以IP没变，还是192.168.1.1。用户名root,密码是上一步获得的密码。

```console
~$ ssh root@192.168.1.1
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
SHA256:TMdKPyzK1rPxHy+/OYVcCsLMVZGEGtpmABnRw1DgDlQ.
Please contact your system administrator.
Add correct host key in ~/.ssh/known_hosts to get rid of this message.
Offending ED25519 key in ~/.ssh/known_hosts:62
  remove with:
  ssh-keygen -f "~/.ssh/known_hosts" -R "192.168.1.1"
RSA host key for 192.168.1.1 has changed and you have requested strict checking.
Host key verification failed.
~$ ssh-keygen -f "~/.ssh/known_hosts" -R "192.168.1.1"
# Host 192.168.1.1 found: line 62
~/.ssh/known_hosts updated.
Original contents retained as ~/.ssh/known_hosts.old
~$ ssh root@192.168.1.1
The authenticity of host '192.168.1.1 (192.168.1.1)' can't be established.
RSA key fingerprint is SHA256:TMdKPyzK1rPxHy+/OYVcCsLMVZGEGtpmABnRw1DgDlQ.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.1.1' (RSA) to the list of known hosts.
root@192.168.1.1's password: 


BusyBox v1.19.4 (2018-06-21 09:07:05 UTC) built-in shell (ash)
Enter 'help' for a list of built-in commands.

 -----------------------------------------------------
       Welcome to XiaoQiang!
 -----------------------------------------------------
  $$$$$$\  $$$$$$$\  $$$$$$$$\      $$\      $$\        $$$$$$\  $$\   $$\
 $$  __$$\ $$  __$$\ $$  _____|     $$ |     $$ |      $$  __$$\ $$ | $$  |
 $$ /  $$ |$$ |  $$ |$$ |           $$ |     $$ |      $$ /  $$ |$$ |$$  /
 $$$$$$$$ |$$$$$$$  |$$$$$\         $$ |     $$ |      $$ |  $$ |$$$$$  /
 $$  __$$ |$$  __$$< $$  __|        $$ |     $$ |      $$ |  $$ |$$  $$<
 $$ |  $$ |$$ |  $$ |$$ |           $$ |     $$ |      $$ |  $$ |$$ |\$$\
 $$ |  $$ |$$ |  $$ |$$$$$$$$\       $$$$$$$$$  |       $$$$$$  |$$ | \$$\
 \__|  \__|\__|  \__|\________|      \_________/        \______/ \__|  \__|


root@XiaoQiang:~# 
```

## 备份

/dev/sda1是U盘，自动mount在/extdisks/sda1。
用dd命令备份到U盘上。

```console
root@XiaoQiang:~# df -h
Filesystem                Size      Used Available Use% Mounted on
rootfs                   10.8M     10.8M         0 100% /
/dev/root                10.8M     10.8M         0 100% /
tmpfs                    61.0M    968.0K     60.0M   2% /tmp
tmpfs                   512.0K         0    512.0K   0% /dev
tmpfs                    61.0M    968.0K     60.0M   2% /extdisks
/dev/mtdblock7            1.0M    772.0K    252.0K  75% /data
/dev/mtdblock7            1.0M    772.0K    252.0K  75% /etc
tmpfs                    61.0M    968.0K     60.0M   2% /userdisk/sysapihttpd
/dev/sda1               116.4G      9.2G    107.2G   8% /extdisks/sda1
/dev/root                 1.0M    772.0K    252.0K  75% /mnt
/dev/mtdblock7            1.0M    772.0K    252.0K  75% /mnt
root@XiaoQiang:~# cat /proc/mtd
dev:    size   erasesize  name
mtd0: 01000000 00010000 "ALL"
mtd1: 00030000 00010000 "Bootloader"
mtd2: 00010000 00010000 "Config"
mtd3: 00010000 00010000 "Factory"
mtd4: 00c80000 00010000 "OS1"
mtd5: 00b178a7 00010000 "rootfs"
mtd6: 00200000 00010000 "OS2"
mtd7: 00100000 00010000 "overlay"
mtd8: 00010000 00010000 "crash"
mtd9: 00010000 00010000 "reserved"
mtd10: 00010000 00010000 "Bdata"
root@XiaoQiang:~# dd if=/dev/mtd0 of=/extdisks/sda1/0-all.bin
32768+0 records in
32768+0 records out
root@XiaoQiang:~# dd if=/dev/mtd1 of=/extdisks/sda1/1-bootloader.bin
384+0 records in
384+0 records out
root@XiaoQiang:~# dd if=/dev/mtd2 of=/extdisks/sda1/2-config.bin
128+0 records in
128+0 records out
root@XiaoQiang:~# dd if=/dev/mtd3 of=/extdisks/sda1/3-Factory.bin
128+0 records in
128+0 records out
root@XiaoQiang:~# dd if=/dev/mtd4 of=/extdisks/sda1/4-OS1.bin
25600+0 records in
25600+0 records out
root@XiaoQiang:~# dd if=/dev/mtd5 of=/extdisks/sda1/5-rootfs.bin
22716+1 records in
22716+1 records out
root@XiaoQiang:~# dd if=/dev/mtd6 of=/extdisks/sda1/6-OS2.bin
4096+0 records in
4096+0 records out
root@XiaoQiang:~# dd if=/dev/mtd7 of=/extdisks/sda1/7-overlay.bin
2048+0 records in
2048+0 records out
root@XiaoQiang:~# dd if=/dev/mtd8 of=/extdisks/sda1/8-crash.bin
128+0 records in
128+0 records out
root@XiaoQiang:~# dd if=/dev/mtd9 of=/extdisks/sda1/9-reserved.bin
128+0 records in
128+0 records out
root@XiaoQiang:~# dd if=/dev/mtd10 of=/extdisks/sda1/10-Bdata.bin
128+0 records in
128+0 records out
root@XiaoQiang:~# 
```

## 写入breed

breed下载

https://breed.hackpascal.net/breed-mt7620-xiaomi-mini.bin

上传breed-mt7620-xiaomi-mini.bin文件并写入路由器。

```console
$ scp breed-mt7620-xiaomi-mini.bin root@192.168.1.1:/tmp
root@192.168.1.1's password: 
breed-mt7620-xiaomi-mini.bin                                                                              100%   88KB 958.9KB/s   00:00    
```

写入

```console
root@XiaoQiang:~# mtd -r write /tmp/breed-mt7620-xiaomi-mini.bin Bootloader
Unlocking Bootloader ...

Writing from /tmp/breed-mt7620-xiaomi-mini.bin to Bootloader ...     
Rebooting ...
```

## openwrt官网固件下载

小米路由器mini的页面 https://openwrt.org/toh/xiaomi/miwifi_mini

下载固件openwrt-22.03.3-ramips-mt7620-xiaomi_miwifi-mini-squashfs-sysupgrade.bin


## 进入breed
拔掉路由器的电源
按住 reset 按钮之后重新接入电源，指示灯变为闪烁状态即可松开 reset 键
有线连接路由器，在浏览器输入 192.168.1.1 进入 Breed 控制台

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/22/xiaomi_mini_2_breed-1677048511940.png)

## 备份原来的固件

点击“固件备份”，备份固件，以防以后想刷回去

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/22/xiaomi_mini_2_breed_backup-1677048556835.png)

## 刷入openwrt固件

点击“固件更新”，选中固件，文件选择我们准备工作中的openwrt固件，点击上传

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/22/xiaomi_mini_2_write-1677048714416.png)

路由器会自动重启并进入openwrt。

## 进入openwrt

连接WAN网线到上级路由或光猫，连接LAN网线到电脑。浏览器打开http://192.168.1.1

默认用户名密码root/password

至此，恭喜你！openwrt刷入成功

## 设置为中继桥和打印服务器

### 安装插件

``` text
1. 中继 (https://openwrt.org/docs/guide-user/network/wifi/relay_configuration)
relayd
luci-proto-relay

2. USB打印机 (https://openwrt.org/docs/guide-user/services/print_server/p910ndprinterserver)
kmod-usb-printer
p910nd
luci-app-p910nd
luci-i18n-p910nd-zh-cn

3. 界面汉化
luci-i18n-base-zh-cn
luci-i18n-firewall-zh-cn
luci-i18n-opkg-zh-cn
```

命令行方式

ssh到路由器后台

```console
ssh root@192.168.1.1
```

更新package

```console
root@OpenWrt:~# opkg update
Downloading https://downloads.openwrt.org/releases/22.03.3/targets/ramips/mt7620/packages/Packages.gz
Updated list of available packages in /var/opkg-lists/openwrt_core
Downloading https://downloads.openwrt.org/releases/22.03.3/targets/ramips/mt7620/packages/Packages.sig
Signature check passed.
Downloading https://downloads.openwrt.org/releases/22.03.3/packages/mipsel_24kc/base/Packages.gz
Updated list of available packages in /var/opkg-lists/openwrt_base
Downloading https://downloads.openwrt.org/releases/22.03.3/packages/mipsel_24kc/base/Packages.sig
Signature check passed.
Downloading https://downloads.openwrt.org/releases/22.03.3/packages/mipsel_24kc/luci/Packages.gz
Updated list of available packages in /var/opkg-lists/openwrt_luci
Downloading https://downloads.openwrt.org/releases/22.03.3/packages/mipsel_24kc/luci/Packages.sig
Signature check passed.
Downloading https://downloads.openwrt.org/releases/22.03.3/packages/mipsel_24kc/packages/Packages.gz
Updated list of available packages in /var/opkg-lists/openwrt_packages
Downloading https://downloads.openwrt.org/releases/22.03.3/packages/mipsel_24kc/packages/Packages.sig
Signature check passed.
Downloading https://downloads.openwrt.org/releases/22.03.3/packages/mipsel_24kc/routing/Packages.gz
Updated list of available packages in /var/opkg-lists/openwrt_routing
Downloading https://downloads.openwrt.org/releases/22.03.3/packages/mipsel_24kc/routing/Packages.sig
Signature check passed.
Downloading https://downloads.openwrt.org/releases/22.03.3/packages/mipsel_24kc/telephony/Packages.gz
Updated list of available packages in /var/opkg-lists/openwrt_telephony
Downloading https://downloads.openwrt.org/releases/22.03.3/packages/mipsel_24kc/telephony/Packages.sig
Signature check passed.
```

安装打印机相关模块

```console
root@OpenWrt:~# opkg install kmod-usb-printer
Installing kmod-usb-printer (5.10.161-1) to root...
Downloading https://downloads.openwrt.org/releases/22.03.3/targets/ramips/mt7620/packages/kmod-usb-printer_5.10.161-1_mipsel_24kc.ipk
Configuring kmod-usb-printer.
root@OpenWrt:~# opkg install p910nd
Installing p910nd (0.97-9) to root...
Downloading https://downloads.openwrt.org/releases/22.03.3/packages/mipsel_24kc/packages/p910nd_0.97-9_mipsel_24kc.ipk
Configuring p910nd.
root@OpenWrt:~# opkg install luci-app-p910nd
Installing luci-app-p910nd (git-20.108.38431-8f34e10) to root...
Downloading https://downloads.openwrt.org/releases/22.03.3/packages/mipsel_24kc/luci/luci-app-p910nd_git-20.108.38431-8f34e10_all.ipk
Installing luci-compat (git-22.069.45071-03bb0e2) to root...
Downloading https://downloads.openwrt.org/releases/22.03.3/packages/mipsel_24kc/luci/luci-compat_git-22.069.45071-03bb0e2_all.ipk
Configuring luci-compat.
Configuring luci-app-p910nd.
root@OpenWrt:~# opkg install luci-i18n-p910nd-zh-cn
Installing luci-i18n-p910nd-zh-cn (git-22.339.56129-deebfa0) to root...
Downloading https://downloads.openwrt.org/releases/22.03.3/packages/mipsel_24kc/luci/luci-i18n-p910nd-zh-cn_git-22.339.56129-deebfa0_all.ipk
Configuring luci-i18n-p910nd-zh-cn.
root@OpenWrt:~# 
```

安装中继桥模块

```console
root@OpenWrt:~# opkg install relayd
Package relayd (2020-04-25-f4d759be-1) installed in root is up to date.
root@OpenWrt:~# opkg install luci-proto-relay
Installing luci-proto-relay (git-19.307.61018-284918b) to root...
Downloading https://downloads.openwrt.org/releases/22.03.3/packages/mipsel_24kc/luci/luci-proto-relay_git-19.307.61018-284918b_all.ipk
Configuring luci-proto-relay.
```

安装汉化模块

```console
root@OpenWrt:~# opkg install luci-i18n-base-zh-cn
Installing luci-i18n-base-zh-cn (git-23.021.35986-09d68fb) to root...
Downloading https://downloads.openwrt.org/releases/22.03.3/packages/mipsel_24kc/luci/luci-i18n-base-zh-cn_git-23.021.35986-09d68fb_all.ipk
Configuring luci-i18n-base-zh-cn.
root@OpenWrt:~# opkg install luci-i18n-firewall-zh-cn
Installing luci-i18n-firewall-zh-cn (git-23.021.35986-09d68fb) to root...
Downloading https://downloads.openwrt.org/releases/22.03.3/packages/mipsel_24kc/luci/luci-i18n-firewall-zh-cn_git-23.021.35986-09d68fb_all.ipk
Configuring luci-i18n-firewall-zh-cn.
root@OpenWrt:~# opkg install luci-i18n-opkg-zh-cn
Installing luci-i18n-opkg-zh-cn (git-23.021.35986-09d68fb) to root...
Downloading https://downloads.openwrt.org/releases/22.03.3/packages/mipsel_24kc/luci/luci-i18n-opkg-zh-cn_git-23.021.35986-09d68fb_all.ipk
Configuring luci-i18n-opkg-zh-cn.
```

重启路由

### 配置无线中继

启动openwrt后，按照这个文章配置无线中继。https://openwrt.org/docs/guide-user/network/wifi/relay_configuration

1. 网络-接口-LAN-编辑 修改路由器为其他网段并关闭DHCP

修改IP为192.168.7.1，另外，还要修改DNS，例如设置成192.168.1.1或者114.114.114.114

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/23/xiaomi_mini_3_openwrt_relay_1-1677114892474.png)

关闭DHCP

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/23/xiaomi_mini_3_openwrt_relay_2-1677114971634.png)

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/23/xiaomi_mini_3_openwrt_relay_3-1677114995555.png)

2. 网络-无线-扫描 加入上级无线网

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/23/xiaomi_mini_3_openwrt_relay_4wifi-1677115052691.png)

无线连接到上级WIFI

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/23/xiaomi_mini_3_openwrt_relay_5wifi-1677115105759.png)

测试一下，看是否连接成功

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/23/xiaomi_mini_3_openwrt_relay_6test-1677115158167.png)

3. 网络-接口 设置刚添加的WWAN接口固定IP

设置固定IP为192.168.1.101

4. 网络-接口-添加 添加中继桥接口

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/23/xiaomi_mini_3_openwrt_relay_8_repeater_bridge-1677115325787.png)

设置IP为192.168.1.101,网络间中继选lan和wwan

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/23/xiaomi_mini_3_openwrt_relay_9_repeater_bridge-1677115478723.png)

到这一步网络就设置好了。路由器是固定IP，连接此路由器的终端会被分配192.168.1.0/24网段的IP。

## 设置打印机服务

打印机连接路由器USB口。

打开路由器上的打印机服务，服务-p910nd打印服务器

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/23/xiaomi_mini_3_openwrt_10_printer-1677115716858.png)

客户端例如Linux上的CUPS上用socket://192.168.1.101:9100连接打印机。

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/23/xiaomi_mini_3_openwrt_10_printer_cups-1677115910687.png)

打印机驱动需要在客户端安装。






