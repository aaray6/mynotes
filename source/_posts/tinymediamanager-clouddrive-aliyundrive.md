---
title: 用tinymediamanager通过clouddrive更新aliyundrive上的视频信息
date: 2023-02-09 23:51:54
tags:
---

在Linux(x64)上用tinymediamanager通过clouddrive更新aliyundrive上的视频信息

电影和电视文件在阿里云盘上，通过clouddrive映射成本地文件，最后通过TMM来更新影片信息。

以上是在Linux(64)环境下做的。如果用Windows，思路相同，但应该更简单。因为TMM和clouddrive本身就是基于Windows的。

在Linux上有以下问题

1. TMM没有arm64版本的。所以只能用x64(amd64)
2. TMM是用Java写的。运行内存要求比较高。通过docker执行还需要vnc和图形界面。所以在1G内存的机器上跑不起来。
3. TMM官方的docker镜像(tinymediamanager/tinymediamanager)支持中文。另外一个(romancin/tinymediamanager)中文显示乱码
4. 在TMM中需要设置代理访问外网的TMDB。
5. 曾经尝试用rclone+aliyunwebdav的方式取代clouddrive映射到本地目录。但是TMM docker启动时访问rclone的目录总是报权限错误。

所以总的来说可以用但效果不好。只是记录思路和过程中用到的参数。

## clouddrive

docker-compose.yml

```yml
version: "2.1"
services:
  cloudnas:
    image: cloudnas/clouddrive
    container_name: clouddrive
    volumes:
      - ~/mnt/CloudNAS:/CloudNAS:shared
      - ~/docker/clouddrive/config:/Config
    devices:
      - /dev/fuse:/dev/fuse
    restart: unless-stopped
    pid: "host"
    privileged: true #or you can try capp_add -SYS_ADMIN
    #cap_add: #SYS_ADMIN cap may fail on some OSes, use privileged: true instead
    # - SYS_ADMIN
    network_mode: "host" #if network_mode doesn't work, use port mapping
    #ports:
    #   - 9798:9798
```

## TMM docker

```console
docker run \
    --name=tinymediamanager \
    -p 4000:4000 \
    -v ~/docker/tinymediamanager/data/:/data \
    -v ~/mnt/CloudNAS/CloudDrive/aliyundrive/01_videos/02_movie/:/media/movies \
    -v ~/mnt/CloudNAS/CloudDrive/aliyundrive/01_videos/01_tv/:/media/tvshows \
    -v ~/docker/tinymediamanager/addons/:/app/addons \
    -e GROUP_ID=0 -e USER_ID=0 -e TZ=Asia/Shanghai \
    tinymediamanager/tinymediamanager:latest
```

## ssh隧道

```console
ssh -L 127.0.0.1:9798:127.0.0.1:9798 -N -f <user>@mypc.theworkpc.com
```

## TMM java版

TMM有java版。https://www.tinymediamanager.org/download/

Linux下TMM需要[libmediainfo](https://mediaarea.net/en/MediaInfo),用如下命令安装

```console
sudo apt install libmediainfo-dev
```

下载之后解压直接运行./tinyMediaManager

## aliyunwebdav+rclone

### aliyundrive-webdav+rclone

aliyundrive-webdav配合rclone

aliyundrive-webdav的配置看[用aliyundrive-webdav在linux下访问阿里云盘](/2022/09/13/aliyundrive-webdav/)

rclone配置看[rclone](/2022/09/15/rclone/)

### TODO,

尝试配合
alist+rclone
clouddrive