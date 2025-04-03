---
title: bgm.tv导出工具bgm_tenkou
date: 2020-05-23 17:28:03
tags:
---

## 获取最新代码

```console
cd ~/Dropbox
git clone git@github.com:aaray6/bgm_tenkou.git bgm_tenkou
```

## 准备环境

```console
cd ~/Dropbox/bgm_tenkou
mkdir aaray
chmod +x *.py
```

aaray是保存导出文件的目录

## Proxychains 代理

编辑~/proxychains.conf,如果没有这个文件，可以拷贝/etc/proxychains.conf到~/proxychains.conf
启动代理监听本地2080或者3080端口
设置~/proxychains.conf文件

```conf
...
#socks5         127.0.0.1 2080
socks5  127.0.0.1 3080
...
```

## 用proxychains启动一个新的终端

此终端可以通过代理连接bgm.tv，速度更快更稳定

```console
proxychains terminator  &
```

## 导出

在terminator中运行

```console
cd ~/Dropbox/bgm_tenkou
./tenkou.py -u <uid> -p ./aaray
```
