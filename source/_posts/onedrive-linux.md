---
title: Linux(ubuntu 18.04)安装OneDrive Free
date: 2020-05-29 17:32:55
tags:
---

## 安装OneDrive Free

> 参考 [如何实现 Microsoft OneDrive 与 Linux 的同步](https://zhuanlan.zhihu.com/p/55622552)
> 软件 [OneDrive Free Client https://github.com/skilion/onedrive](https://github.com/skilion/onedrive)

按照https://github.com/skilion/onedrive的文档安装。

sudo apt install libcurl4-openssl-dev
sudo apt install libsqlite3-dev
sudo snap install --classic dmd && sudo snap install --classic dub

systemctl status snapd.service
systemctl start snapd.service

git clone https://github.com/skilion/onedrive.git
cd onedrive
make
sudo make install

onedrive

~/OneDrive目录是默认目录

OneDrive service
systemctl --user enable onedrive
systemctl --user start onedrive
journalctl --user-unit onedrive -f

## 卸载Dropbox

> 参考[Linux系统怎么卸载软件? Linux卸载Dropbox的教程](https://www.jb51.net/LINUXjishu/521822.html)

用以下命令卸载
sudo apt-get autoremove --purge dropbox 

```console
$ sudo apt-get autoremove --purge dropbox 
[sudo] xxxx 的密码： 
正在读取软件包列表... 完成
正在分析软件包的依赖关系树       
正在读取状态信息... 完成       
下列软件包将被【卸载】：
  dropbox* gir1.2-geocodeglib-1.0* gtk3-nocsd* libfwup1* libgtk3-nocsd0*
  libllvm7* libllvm7:i386* libllvm8* libllvm8:i386* libsystemd0:i386*
  libudev1:i386* shimmer-themes* ubuntu-web-launchers* xubuntu-wallpapers*
升级了 0 个软件包，新安装了 0 个软件包，要卸载 14 个软件包，有 39 个软件包未被升级。
解压缩后将会空出 261 MB 的空间。
您希望继续执行吗？ [Y/n] Y
(正在读取数据库 ... 系统当前共安装有 328730 个文件和目录。)
正在卸载 dropbox (2019.01.31) ...
正在卸载 ubuntu-web-launchers (18.04.7) ...
正在卸载 gir1.2-geocodeglib-1.0:amd64 (3.25.4.1-4ubuntu0.18.04.1) ...
......
(正在读取数据库 ... 系统当前共安装有 328638 个文件和目录。)
正在清除 gtk3-nocsd (3-1ubuntu1) 的配置文件 ...
正在清除 dropbox (2019.01.31) 的配置文件 ...
```
