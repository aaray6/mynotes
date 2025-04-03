---
title: 用aliyundrive-webdav在linux下访问阿里云盘
date: 2022-09-13 13:40:48
tags:
---

用aliyundrive-webdav在linux下访问阿里云盘

> https://github.com/messense/aliyundrive-webdav

首先需要有阿里云盘，然后可以通过以上程序把阿里云盘分享为webdav服务，并通过ubuntu下的文件管理器访问。或者通过kodi在电视盒子上直接访问里面的视频等。

安装方法参考上面链接。

可以安装在x86, arm64(N1)等系统下。可以本地安装，也可以用docker运行。

## N1(arm64 armbian)下用docker运行

```console
$docker pull messense/aliyundrive-webdav
$mkdir ~/docker/aliyundrive-webdav
$cd ~/docker/aliyundrive-webdav
$cat docker-compose.yml

version: '3.3'
services:
  aliyundrive-webdav:
    container_name: aliyundrive-webdav
    restart: unless-stopped
    ports:
      - '8088:8080'
    environment:
      - 'REFRESH_TOKEN=28d5...5d17f'
    image: messense/aliyundrive-webdav

$docker-compose up
Creating network "aliyundrivewebdav_default" with the default driver
Creating aliyundrive-webdav ... 
Creating aliyundrive-webdav ... done
Attaching to aliyundrive-webdav
aliyundrive-webdav    | 2022-09-13T01:17:42.581871Z  INFO aliyundrive_webdav::drive: refresh token succeed refresh_token=22e0...25fd nick_name=#####
aliyundrive-webdav    | 2022-09-13T01:17:42.588252Z  INFO aliyundrive_webdav::drive: found default drive drive_id=nnnnnnnnn
aliyundrive-webdav    | 2022-09-13T01:17:42.589031Z  INFO aliyundrive_webdav: listening on http://0.0.0.0:8080
```

此时webdav在N1上启动，并监听8088端口，用户名和密码默认是admin/admin。

### 通过ubuntu文件管理器（nautilus）访问

在文件管理器中点“+其他位置”，在“连接到服务器”后面添“dav://admin@192.168.1.23:8088”，然后点连接即可。就可以跟访问本地磁盘一样访问阿里云盘。

### 通过KODI访问

KODI支持webdav协议，在KODI中可以添加192.168.1.23:8088，然后就可以直接在KODI中访问云盘内容，不需要下载到本地。

## x86 Linux下用aliyundrive-webdav

在x86 Linux下跟在arm64 armbian下一样。
除了以上方法，还可以不用docker，直接用以下命令运行。

```console
$sudo snap install aliyundrive-webdav
$aliyundrive-webdav -p 8088
```

以上命令执行后webdav在Linux上启动，并监听8088端口，用户名和密码默认是admin/admin。
注:推荐还是用docker

## 尝试用davfs2异常，用rclone可以成功mount webdav

有文章介绍可以同davfs2 mount webdav。但mount之后虽然可以成功显示阿里云盘的内容，但是下载失败。
还是记录以下命令，以后有机会再试试。
在用aliyundrive-webdav把阿里云盘分享为webdav服务之后，做以下操作。

```console
sudo apt-get install davfs2
sudo mount -t davfs -o noexec,uid=1000,gid=1000 http://127.0.0.1:8088 /media/dav/
```

连上之后，甚至可以成功在/media/dav目录下执行echo 123 > 123.txt命令。
md5sum 123.txt
cat 123.txt
都可以。

但是过几秒钟后md5sum/cat命令就不行了。报input/output error。

用rclone可以mount,具体方法参考另一个记录[rclone](/2022/09/15/rclone/)

## 在Linux下用小白羊下载阿里云盘的文件

小白羊支持win/mac/linux,我在linux下用的是x86的

> https://github.com/liupan1890/aliyunpan

2022.9.13,以上github里最新的release是2.9.24，但是这个版本已经无法登录阿里云盘。

需要到https://wwe.lanzoui.com/b01nqc4gd里下载2.12.14版。

小白羊可以设置使用其他服务器上的aria2下载，比如在N1上用docker上安装了aria2，在linux下启动小白羊，可以连接到N1的aria2，方法如下。
![本地图片](/myimages/xiaobaiyang_1.png "小白羊设置远程Aria2")
注意的是，在"Aria2远程文件下载保存位置"里，添的是docker中的目录"/downloads"而不是host的目录"/media/nas001/downloads"。
