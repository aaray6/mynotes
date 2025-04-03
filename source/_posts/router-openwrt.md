---
title: 路由器openwrt相关
date: 2022-10-26 12:40:14
tags:
---

路由器无线组网相关记录。

## 硬件

1. Netgear R6220
2. 小米R3G

两个路由都是MT7621,128M Flash, 256M内存。都有breed。都可以刷padavan, openwrt。
R6220是1千兆WAN+4千兆LAN+1USB2.0，2.4G+5G双频1200M。准备接USB打印机做打印服务器。
小米R3G是1千兆WAN+2千兆LAN+1USB3.0，2.4G+5G双频1200M。准备接移动硬盘做文件服务器。

## 网络拓扑

TODO

## 刷breed

### R3G

刷breed的方法参考https://www.cnblogs.com/milton/p/16163521.html

breed下载网址https://breed.hackpascal.net/
对应R3G的文件是breed-mt7621-xiaomi-r3g.bin

### R6220

根据网上的文档https://opssh.cn/luyou/205.html安装breed。
breed下载网址https://breed.hackpascal.net/
对应R6220的文件是breed-mt7621-r6220.bin

## 无线中继方案

主路由R3G,接移动硬盘，USB3.0速度可以达到读90+MB/s,写40+MB/s。
主路由可以选择官方固件或者其他Padavan, openwrt。

### 主路由R3G

理论上可以选择任何固件。

R3G官方固件高级设置里有漫游功能。但不知道是否是802.11k/v/r协议。所以没选。

Padavan固件4.x.100版本中有k/v/r支持。还支持USB3.0，网络共享，科学上网。所以选用此固件。

Openwrt官方待测。

### 副路由R6220

副路由R6220必须是openwrt，官方openwrt固件就可以。因为需要的中继relayd, 打印服务p910nd都可以作为插件安装。

副路由为什么不选官方固件和Padavan?
官方固件有中继模式，可以中继，而且很方便。但是选择中继模式后文件服务器或者打印服务器都不能再用，USB口相当于浪费了。
Padavan目前固件也可以中继。但是没有打印服务器功能。R3G的Padavan固件版本新，可以做打印服务器。但是中继时路由器无法指定固定IP，作为打印服务器或者文件服务器都不方便。

R6220安装官方openwrt固件并安装插件。

安装breed后，同时按wps键和开关键开机，大概4秒后电源指示灯闪几下，网线连接WAN口，浏览器访问192.168.1.1就可以打开breed界面。

然后下载官网固件22.03.1 (截至2022.10)
https://openwrt.org/toh/netgear/r6220

https://downloads.openwrt.org/releases/22.03.2/targets/ramips/mt7621/openwrt-22.03.2-ramips-mt7621-netgear_r6220-squashfs-factory.img是在breed中用的刷机固件。

https://downloads.openwrt.org/releases/22.03.2/targets/ramips/mt7621/openwrt-22.03.2-ramips-mt7621-netgear_r6220-squashfs-sysupgrade.bin是在openwrt中升级的固件。

安装插件

```text
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

4. 阿里云webdav (https://github.com/messense/aliyundrive-webdav)
wget https://github.com/messense/aliyundrive-webdav/releases/download/v1.10.1/aliyundrive-webdav_1.10.1-1_mipsel_24kc.ipk
wget https://github.com/messense/aliyundrive-webdav/releases/download/v1.10.1/luci-app-aliyundrive-webdav_1.10.1_all.ipk
wget https://github.com/messense/aliyundrive-webdav/releases/download/v1.10.1/luci-i18n-aliyundrive-webdav-zh-cn_1.10.1-1_all.ipk
opkg install aliyundrive-webdav_1.10.1-1_aarch64_generic.ipk
opkg install luci-app-aliyundrive-webdav_1.10.1_all.ipk
opkg install luci-i18n-aliyundrive-webdav-zh-cn_1.10.1-1_all.ipk
```

启动openwrt后，按照这个文章配置无线中继。https://openwrt.org/docs/guide-user/network/wifi/relay_configuration

配置时注意的要点

1. 副路由的IP必须是另一个网段(192.168.3.1)。
2. 必须关闭副路由的DHCP，包括IPV6。关闭后，电脑连路由器必须改成静态IP192.168.3.2。
3. relayd相关的插件如下
kmod-usb-printer
p910nd
luci-app-p910nd
luci-i18n-p910nd-zh-cn

注：目前我没有用上面的官方固件。用的是自编译的固件，把relayd, p910nd,samba和easymesh等相关功能都编译进去了。路由器重置也不需要重新安装插件。

## easymesh方案 (因为R3G自编译带easymesh的openwrt不稳定，放弃easymesh)

这个方案需要主副路由都刷openwrt，而且必须是自己编译的openwrt。因为必须用到easymesh。而easymesh不能作为插件安装，必须编译到内核中，所以只能选择自编译。

注: (2022.10.26)主路由R3G的自编译openwrt使用期间发生过5G wifi速率降到14Mb/s。必须重启才能恢复。而且连续发生两天。路由器最重要的是稳定。不知道是否是easymesh引起的问题。暂时放弃此方案。改用官方固件无线中继模式。

### 编译R3G openwrt

选择LEDE(https://github.com/coolsnowwolf/lede)，这个版本基于openwrt,而且集成了easymesh和其他有用的插件，更方便。

编译环境: Ubuntu 18.04.6 LTS (Bionic Beaver)
磁盘空间大约需要70G.

具体步骤按照https://github.com/coolsnowwolf/lede页面说明的做。

步骤就是文档中提到的，编译我选择2或3线程。选择1线程需要编译4个小时以上。

make V=s -j2

我的目标是在默认配置下，做如下更改

```text
+磁盘管理system/diskman
-上网时间控制 services/mia
-动态DNSservices/ddns

+automount
+打印服务器 nas/p910nd
+FTP服务器nas/vsftpd
+网络共享nas/samba

+简单MESH network/easymesh
```

在make menuconfig那一步，我按照如下选择模块。这个选择加入如下功能
1. USB3.0文件存储支持。支持常见的几种文件系统(NTFS, FAT, FAT32, ext4等)

2. 磁盘管理，可以自动mount U盘。

3. easymesh

4. 打印服务器

5. 文件共享

6. 访客网络

7. 终端

8. 加了两个主题，增加了英语

9. 中继模块

```text
Target System (MediaTek Ralink MIPS)
Subtarget (MT7621 based boards)
Target Profile (Xiaomi Mi Router 3G)

LuCI->Applications
+ luci-app-diskman......................... Disk Manager interface for LuCI
+ Include lsblk
+ luci-app-easymesh.............................. LuCI Support for easymesh
+ luci-app-guest-wifi.......................... LuCI support for guest-wifi
+ luci-app-p910nd........................... p910nd - Printer server module
+ luci-app-samba.................... Network Shares - Samba SMB/CIFS module
+ luci-app-ttyd...................................... LuCI support for ttyd
- luci-app-vlmcsd..................................... LuCI support for KMS
- luci-app-wol................................ LuCI Support for Wake-on-LAN

LuCI->Themes
+ luci-theme-material....................................... Material Theme
+ luci-theme-netgear......................................... Netgear Theme

LuCI->Protocols
+ luci-proto-ipv6........... Support for DHCPv6/6in4/6to4/6rd/DS-Lite/aiccu
+ luci-proto-relay....................... Support for relayd pseudo bridges

LuCI->Modules -> Translations
+ English (en)

Extra packages
+ automount............................... Mount autoconfig hotplug script (这个选择后会自动选择Kernel modules->Filesystems下的很多模块，如果需要支持更多的文件系统，需要在里面增加相应模块)

Kernel modules->USB Support
+ kmod-usb-storage-uas.................... USB Attached SCSI (UASP) support (选择这个才能让R3G路由器支持USB3.0，否则读写速度只有20多M，打开后读可以90+M,写可以40+M)
+ kmod-usb-printer.................................... Support for printers (打印机需要)
```

如果对配置不满意，可以重新配置
删除.config
cd ~/dev/openwrt/lede
git pull （这个目的是为了更新代码）
rm .config
make menuconfig

### 编译R6220 openwrt

因为R6220和R3G用的CPU都是MT7621,所以配置几乎一样。唯一的区别是

```text
Target System (MediaTek Ralink MIPS)
Subtarget (MT7621 based boards)
Target Profile (Netgear R6220)
```

#### R6220 openwrt 2.4G丢失问题

> [mt7621: R6220 2.4GHz radio0 issues
](https://github.com/openwrt/openwrt/issues/9374)

"wifiseguro"的回答

Just add a file in patches folder ie:
target/linux/ramips/patches-5.10/901-staging-mt7621-pci-delay-for-properly-detect.patch

```patch
--- a/drivers/staging/mt7621-pci/pci-mt7621.c
+++ b/drivers/staging/mt7621-pci/pci-mt7621.c
@@ -502,8 +502,9 @@
 	int err;
 
 	rt_sysc_m32(PERST_MODE_MASK, PERST_MODE_GPIO, MT7621_GPIO_MODE);
-
+	mdelay(250);
 	mt7621_pcie_reset_assert(pcie);
+	mdelay(250);
 	mt7621_pcie_reset_rc_deassert(pcie);
 
 	list_for_each_entry_safe(port, tmp, &pcie->ports, list) {
@@ -520,7 +521,7 @@
 			list_del(&port->list);
 		}
 	}
-
+	mdelay(250);
 	mt7621_pcie_reset_ep_deassert(pcie);
 
 	tmp = NULL;
```

我找到了3个带patches-5xxx的目录

```console
quxr@quxr-ThinkPad-X201:~/dev/openwrt/lede$ find . -name "patches-5*"|grep ramips
./target/linux/ramips/patches-5.15
./target/linux/ramips/patches-5.10
./target/linux/ramips/patches-5.4
```

我认为应该加到patches-5.4,因为R6220用的是这个目录。但是为了稳妥起见，我在三个目录都加了。

```console
quxr@quxr-ThinkPad-X201:~/dev/openwrt/lede$ find . -name "901-staging-mt7621-pci-delay-for-properly-detect.patch"
./target/linux/ramips/patches-5.15/901-staging-mt7621-pci-delay-for-properly-detect.patch
./target/linux/ramips/patches-5.10/901-staging-mt7621-pci-delay-for-properly-detect.patch
./target/linux/ramips/patches-5.4/901-staging-mt7621-pci-delay-for-properly-detect.patch
```

然后重新编译就可以了。
从patch看，就是加了几处延时。但的确立即解决2.4G找不到的问题。

## 旁路由网关

主路由是R3G刷openwrt, 192.168.1.1
用ZTE E8820S刷openwrt无线中继，接打印机, 192.168.1.101
N1装armbian接在R3G,192.168.1.108
  N1下docker运行openwrt做旁路由网关192.168.1.111
CM311-1sa装armbian接在E8820S下,192.168.1.106
  CM311-1sa下docker运行openwrt做另外一个旁路由网关192.168.1.112

电脑x201通过wifi接入R3G上网。

```console
# 设置192.168.1.111为默认网关
sudo route add default gw 192.168.1.111 wlp2s0
# 删除192.168.1.111网关
sudo route delete default gw 192.168.1.111 wlp2s0

# 设置192.168.1.112为默认网关
sudo route add default gw 192.168.1.112 wlp2s0
# 删除192.168.1.112网关
sudo route delete default gw 192.168.1.112 wlp2s0
```

症状:一开始旁路由能工作，但是不知道什么原因就不工作了。排除了host接在主路由或者无线桥接的可能。因为无论接在哪都一样。有时候好用有时候不好用。

根据这篇文章
> https://blog.csdn.net/qq1337715208/article/details/122271608
> https://cloud.tencent.com/developer/article/2036952

最后，旁路由不需要添加防火墙“iptables -t nat -I POSTROUTING -j MASQUERADE”，只在主路由R3G里做如下修改

```console
root@R3G:/etc# echo 'net.bridge.bridge-nf-call-ip6tables=0' >> /etc/sysctl.conf
root@R3G:/etc# echo 'net.bridge.bridge-nf-call-iptables=0' >> /etc/sysctl.conf 
root@R3G:/etc# echo 'net.bridge.bridge-nf-call-arptables=0' >> /etc/sysctl.conf
root@R3G:/etc# sysctl -p /etc/sysctl.conf 
net.bridge.bridge-nf-call-ip6tables = 0
net.bridge.bridge-nf-call-iptables = 0
net.bridge.bridge-nf-call-arptables = 0
```

然后根据上面的命令使用旁路由为默认网关就可以上网了。

其他记录

在106,108上创建docker openwrt

```console
sudo ip link set eth0 promisc on
docker network create -d macvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1 -o parent=eth0 macnet
docker pull registry.cn-shanghai.aliyuncs.com/suling/openwrt:armv8
docker run --restart always --name openwrt -d --network macnet --privileged registry.cn-shanghai.aliyuncs.com/suling/openwrt:armv8 /sbin/init

docker exec -it openwrt bash
```

或者

```console
sudo ip link set eth0 promisc on
docker network create -d macvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1 -o parent=eth0 macnet108
docker run --restart always -d --name=openwrt --network macnet108 --privileged unifreq/openwrt-aarch64 /sbin/init

docker exec -it openwrt bash
```
