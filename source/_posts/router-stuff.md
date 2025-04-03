---
title: 路由器相关
date: 2022-10-03 11:32:11
tags:
---

路由器相关的记录

## 无线桥接/有线桥接

路由器信号穿墙信号弱的位置，在合适的位置增加路由器，以便增强信号。

### TPLINK WR740N无线路由器怎么实现WDS无线桥接
> TPLINK WR740N无线路由器怎么实现WDS无线桥接(https://zhidao.baidu.com/question/332477764404619205.html)

>[TL-WR740N] 无线桥接（WDS）如何设置(https://resource.tp-link.com.cn/pc/docCenter/showDoc?id=1655112496577635)

上面的链接来自TPLink的官方，尽管步骤差不多，但是官方是要求关闭副路由器的DHCP。我尝试关闭之后连接副路由器的WIFI失败。必须像第一个文章那样开启DHCP才行。

两个路由通过无线桥接之后，其他设备可以连接副路由WIFI或者网口访问网络。

注：家里的设置如下
主路由NETGEAR R6220
LAN:
  IP: 192.168.1.1
  DHCP开(192.168.1.2-192.168.1.199) 
  2.4G WIFI (MYWIFI) 
  5G WIFI(MYWIFI-5G)
WAN: 192.168.2.3
 GATEWAY: 192.168.2.1

副路由1 TPLINK WR740N
LAN:
 IP: 192.168.1.201
 DHCP开(192.168.1.202-192.168.1.209)
 GATEWAY:192.168.1.1
 2.4G WIFI(MYWIFI_1)
 WAN未连接

副路由2 TPLINK  WR340G
LAN:
 IP:192.168.1.211
 DHCP关
 2.4G WIFI(MYWIFI_2)
 WAN未连接

注:主路由器是NETGEAR R6220，最初WR740N的固件版本是
4.18.29 Build 110909 Rel.35946n
可以根据以上方法配置成功。但是运行一段时间后反应变慢。
于是我升级到
5.3.1 Build 130403 Rel64202n
5.3.16 Build 130415 Rel.68098n
后来降级到
5.1.3 Build 121024 Rel.39918n
后面的几个版本都无法做到无线桥接。尽管WDS状态显示成功。但是只能ping到副路由器，无法ping主路由器。

最后我用5.1.3 Build 121024 Rel.39918n版本的740N,桥接到WR340G的无线信号。WR340G是有线桥接到R6220上的。也就是两个TPLink的路由之间做无线桥接。这个方法可以成功。
看了TPLink官方文档中提到不保证与其他品牌的路由器桥接是有道理的。

### TPLINK WR340G无线路由器有线桥接

> 路由器有线桥接设置教程(https://www.bilibili.com/read/mobile?id=13497446)

参考上面链接的“方式二、把B路由器作为无线交换机”

1. 副路由器连接电脑，改IP防止冲突，关闭DHCP。设置无线网络和无线网络安全选项。重启。
2. 网线连接副路由器的LAN口和主路由器的LAN口。

电脑连接副路由器的WIFI或者LAN口即可上网。

与之前的最大区别是两个路由器是有线连接。副路由器关闭DHCP。

注：文章中的“方式一、将B路由器设置设置成“动态IP上网””其实就是家里主路由器连接联通路由器的方式。
WR740N用此方法也可以有线桥接。

### NETGEAR路由器的无线桥接

> 如何在netgear网件上设置桥接模式(http://www.goooogl.com.cn/help/2489.html)

注: 我R6220的固件版本V1.1.0.110_1.0.1，没有文章中提到的功能“use other operation mode”。
文章中截图用的是R6300。
新版本固件V1.1.0.114也不能。
