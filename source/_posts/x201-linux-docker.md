---
title: IBM X201下Linux用Docker
date: 2021-10-19 12:29:43
tags:
---

## Linux mount SMB

> <https://unix.stackexchange.com/questions/68079/mount-cifs-network-drive-write-permissions-and-chown>

```console
sudo mount -t cifs -o username=${USER},password=${PASSWORD},uid=$(id -u),gid=$(id -g) //server-address/folder /mount/path/on/ubuntu
```

## openwrt

**无线网卡无法使用macvlan和ipvlan**
无线网络运行docker-openwrt没成功
以下部分不完整，未验证

首先查看网络

```console
$ ifconfig
br-b4a98747b3d2: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.18.0.1  netmask 255.255.0.0  broadcast 172.18.255.255
        inet6 fe80::42:ceff:fe2a:13e9  prefixlen 64  scopeid 0x20<link>
        ether 02:42:ce:2a:13:e9  txqueuelen 0  (以太网)
        RX packets 507008  bytes 598152743 (598.1 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 900628  bytes 368046284 (368.0 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

br-e2eb145b525d: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.20.0.1  netmask 255.255.0.0  broadcast 172.20.255.255
        inet6 fe80::42:77ff:fef8:9824  prefixlen 64  scopeid 0x20<link>
        ether 02:42:77:f8:98:24  txqueuelen 0  (以太网)
        RX packets 672  bytes 3081824 (3.0 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 4084  bytes 881444 (881.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:47ff:fe9f:eb0e  prefixlen 64  scopeid 0x20<link>
        ether 02:42:47:9f:eb:0e  txqueuelen 0  (以太网)
        RX packets 19585  bytes 24649955 (24.6 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 23642  bytes 3295804 (3.2 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

enp0s25: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether 00:26:2d:f9:97:92  txqueuelen 1000  (以太网)
        RX packets 830414  bytes 1157347303 (1.1 GB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 235401  bytes 173530314 (173.5 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 20  memory 0xf2500000-f2520000  

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (本地环回)
        RX packets 2176189  bytes 596722322 (596.7 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2176189  bytes 596722322 (596.7 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

veth5292288: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::fc13:2cff:fe70:ad01  prefixlen 64  scopeid 0x20<link>
        ether fe:13:2c:70:ad:01  txqueuelen 0  (以太网)
        RX packets 13706  bytes 19062757 (19.0 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 17688  bytes 2191631 (2.1 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

veth3bed11c: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::88c2:feff:fee2:76de  prefixlen 64  scopeid 0x20<link>
        ether 8a:c2:fe:e2:76:de  txqueuelen 0  (以太网)
        RX packets 113318  bytes 46986357 (46.9 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 167254  bytes 77382022 (77.3 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

veth518a5a7: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::fc87:e0ff:fefb:e7c6  prefixlen 64  scopeid 0x20<link>
        ether fe:87:e0:fb:e7:c6  txqueuelen 0  (以太网)
        RX packets 1164  bytes 2177248 (2.1 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1453  bytes 181982 (181.9 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

veth5b903ef: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::a8df:17ff:fe5c:5b28  prefixlen 64  scopeid 0x20<link>
        ether aa:df:17:5c:5b:28  txqueuelen 0  (以太网)
        RX packets 672  bytes 3091232 (3.0 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 4233  bytes 903044 (903.0 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vethb4a1a9c: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::c0d5:4aff:fe26:1c13  prefixlen 64  scopeid 0x20<link>
        ether c2:d5:4a:26:1c:13  txqueuelen 0  (以太网)
        RX packets 172  bytes 263009 (263.0 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 4993  bytes 980000 (980.0 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlp2s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.6  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::c756:b68:e7d3:95ee  prefixlen 64  scopeid 0x20<link>
        ether 00:23:14:31:7e:6c  txqueuelen 1000  (以太网)
        RX packets 2914595  bytes 2196902998 (2.1 GB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2495389  bytes 1890928065 (1.8 GB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

这里wlp2s0是我的无线网卡

用如下命令创建macvlan

```console
docker network create -d macvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1 -o parent=enp0s25 macnet
```

开启网卡混杂模式(这一步不是必须的)

```console
$ sudo ip link set enp0s25 promisc on 
```

下载镜像

> <https://hub.docker.com/r/openwrt/rootfs>

```console
docker pull openwrt/rootfs
```

运行容器

```console
docker run \
        --name openwrt \
        --restart always \
        --network macnet \
        --mac-address  "FC:7C:02:01:02:03" \
        --ip "192.168.1.31" \
        -d \
        --privileged=true \
        openwrt/rootfs
```

进入容器

```console
docker exec -it openwrt /bin/sh
```

N1上的unifreq/openwrt-aarch64上的/etc/config/network文件内容

```config
# cat network 

config interface 'loopback'
 option ifname 'lo'
 option proto 'static'
 option ipaddr '127.0.0.1'
 option netmask '255.0.0.0'

config globals 'globals'
 option ula_prefix 'fd4f:d115:6b60::/48'

config interface 'lan'
 option type 'bridge'
 option ifname 'eth0'
 option proto 'static'
 option ipaddr '192.168.1.31'
 option netmask '255.255.255.0'
 option ip6assign '60'
 option gateway '192.168.1.1'
 option dns '8.8.8.8'

config interface 'VPN'
 option ifname 'ipsec0'
 option proto 'static'
 option ipaddr '10.10.10.1'
 option netmask '255.255.255.0'

config interface 'vpn0'
 option ifname 'tun0'
 option proto 'none'


# ifconfig
br-lan    Link encap:Ethernet  HWaddr FC:7C:02:01:02:03  
          inet addr:192.168.1.31  Bcast:192.168.1.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:655 errors:0 dropped:0 overruns:0 frame:0
          TX packets:469 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:144428 (141.0 KiB)  TX bytes:49534 (48.3 KiB)

eth0      Link encap:Ethernet  HWaddr FC:7C:02:01:02:03  
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:668 errors:0 dropped:0 overruns:0 frame:0
          TX packets:471 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:158590 (154.8 KiB)  TX bytes:51620 (50.4 KiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:132 errors:0 dropped:0 overruns:0 frame:0
          TX packets:132 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:13162 (12.8 KiB)  TX bytes:13162 (12.8 KiB)

```

N1的网络配置

## 尝试用无线网络在virtualbox里运行Debian11，在里面启动openwrt

没成功

virtualbox设置bridged网络，host可以与guest Debian11互相通信。但是用macvlan的Docker openwrt无法与其他服务器通信。

用macvlan无法在host和guest之间通信，这是macvlan本身的特点。
