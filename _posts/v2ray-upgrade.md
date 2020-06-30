---
title: v2ray升级
date: 2020-06-04 11:49:41
tags:
---

v2ray最近几天放出了几个v2ray core release,根据说明，4.23.2之前的版本有安全漏洞，所以需要升级。

## Linux 客户端升级v2raycore

查看v2ray当前版本

```console
/usr/bin/v2ray --version
```

下载最新版本(今天最新是4.23.4)

https://github.com/v2ray/v2ray-core/releases

比如我的ubuntu就可以下载https://github.com/v2ray/v2ray-core/releases/download/v4.23.4/v2ray-linux-64.zip

解压得到以下文件

```console
-rw-r--r-- 1 xxxx xxxx     4147 1月   1  2010 config.json
drwxr-xr-x 2 xxxx xxxx     4096 1月   1  2010 doc
-rw-r--r-- 1 xxxx xxxx  5151626 1月   1  2010 geoip.dat
-rw-r--r-- 1 xxxx xxxx   274688 1月   1  2010 geosite.dat
drwxr-xr-x 2 xxxx xxxx     4096 1月   1  2010 systemd
drwxr-xr-x 2 xxxx xxxx     4096 1月   1  2010 systemv
-r-xr-xr-x 1 xxxx xxxx 11030528 1月   1  2010 v2ctl
-r-xr-xr-x 1 xxxx xxxx        0 1月   1  2010 v2ctl.sig
-r-xr-xr-x 1 xxxx xxxx 16506880 1月   1  2010 v2ray
-r-xr-xr-x 1 xxxx xxxx        0 1月   1  2010 v2ray.sig
-rw-r--r-- 1 xxxx xxxx      391 1月   1  2010 vpoint_socks_vmess.json
-rw-r--r-- 1 xxxx xxxx      537 1月   1  2010 vpoint_vmess_freedom.json
```

停止v2ray service

```console
sudo systemctl stop v2ray
```

把这4个文件

```console
v2ray
v2ctl
geoip.dat
geosite.dat
```

拷贝到/usr/bin/v2ray目录中覆盖同名文件

启动v2ray service

```console
sudo systemctl start v2ray
```

## Windows客户端升级

我用的是v2rayN客户端
在菜单里有“检查更新”->"检查更新v2rayCore"

## 服务器更新

我用的是以下脚本安装的服务器

bash <(curl -L -s https://raw.githubusercontent.com/wulabing/V2Ray_ws-tls_bash_onekey/master/install.sh) | tee v2ray_ins.log

以root身份重新执行这个脚本

在弹出的菜单中选3更新v2rayCore即可。脚本会自动下载更新重启v2ray
