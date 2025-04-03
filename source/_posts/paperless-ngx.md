---
title: paperless-ngx
date: 2023-01-28 22:05:59
tags:
---

Paperless-ngx服务器搭建。

## 安装

> https://docs.paperless-ngx.com/setup/

### 用脚本安装

先安装docker和docker-compose,再执行以下命令开始安装。

``` console
bash -c "$(curl -L https://raw.githubusercontent.com/paperless-ngx/paperless-ngx/main/install-paperless-ngx.sh)"
```

我的安装目录如下
~/docker/paperless-ngx
此目录下放docker-compose.yml，docker-compose.env文件和consume， export目录

~/paperless-ngx
此目录下
data
media

### 手工配置

目录同上，创建如下文件

docker-compose.yml

``` yml
# docker-compose file for running paperless from the Docker Hub.
# This file contains everything paperless needs to run.
# Paperless supports amd64, arm and arm64 hardware.
#
# All compose files of paperless configure paperless in the following way:
#
# - Paperless is (re)started on system boot, if it was running before shutdown.
# - Docker volumes for storing data are managed by Docker.
# - Folders for importing and exporting files are created in the same directory
#   as this file and mounted to the correct folders inside the container.
# - Paperless listens on port 8000.
#
# In addition to that, this docker-compose file adds the following optional
# configurations:
#
# - Instead of SQLite (default), PostgreSQL is used as the database server.
#
# To install and update paperless with this file, do the following:
#
# - Copy this file as 'docker-compose.yml' and the files 'docker-compose.env'
#   and '.env' into a folder.
# - Run 'docker-compose pull'.
# - Run 'docker-compose run --rm webserver createsuperuser' to create a user.
# - Run 'docker-compose up -d'.
#
# For more extensive installation and update instructions, refer to the
# documentation.

version: "3.4"
services:
  broker:
    image: docker.io/library/redis:7
    restart: unless-stopped
    volumes:
      - redisdata:/data

  db:
    image: docker.io/library/postgres:13
    restart: unless-stopped
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: paperless
      POSTGRES_USER: paperless
      POSTGRES_PASSWORD: paperless

  webserver:
    image: ghcr.io/paperless-ngx/paperless-ngx:latest
    restart: unless-stopped
    depends_on:
      - db
      - broker
    ports:
      - 8100:8000
    healthcheck:
      test: ["CMD", "curl", "-fs", "-S", "--max-time", "2", "http://localhost:8000"]
      interval: 30s
      timeout: 10s
      retries: 5
    volumes:
      - ~/paperless-ngx/data:/usr/src/paperless/data
      - ~/paperless-ngx/media:/usr/src/paperless/media
      - ./export:/usr/src/paperless/export
      - ~/docker/paperless-ngx/consume:/usr/src/paperless/consume
    env_file: docker-compose.env
    environment:
      PAPERLESS_REDIS: redis://broker:6379
      PAPERLESS_DBHOST: db


volumes:
  pgdata:
  redisdata:

```

docker-compose.env

``` ini
PAPERLESS_TIME_ZONE=Asia/Shanghai
PAPERLESS_URL=https://<domain_name>
USERMAP_UID=1000
USERMAP_GID=1000
PAPERLESS_OCR_LANGUAGE=chi_sim
PAPERLESS_SECRET_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
PAPERLESS_OCR_LANGUAGES=chi-sim
```

然后用以下命令启动

``` console
docker-compose up -d
```

停止

```console
docker-compose down
```

## 其他命令

### 设置管理员帐号密码

```console
docker-compose run --rm -e DJANGO_SUPERUSER_PASSWORD="<password>" webserver createsuperuser --noinput --username "<username>" --email "<email>"
```

### 导入导出

"-z"参数是压缩，生成的zip文件在~/docker/paperless-ngx/export下。

``` console
docker-compose exec -T webserver document_exporter ../export -z
docker-compose exec -T webserver document_importer ../export -z
```

## Caddy反向代理

在Caddyfile文件中加入如下内容

```yml
<domain_name> {
    proxy / http://10.0.0.32:8100 {
        transparent
    }
}
```

10.0.0.32是网卡内网IP.可以用ip -a命令查询。

配置完成重启Caddy,然后就可以通过 https://<domain_name> 来访问了。并且，端口8100不需要开防火墙。

## 备注

1. 2023.1,最新版本只在amd64下成功运行。在arm64上失败。

arm64上报如下错误

``` log
Apply database migrations...
/sbin/docker-prepare.sh: line 71:   466 Segmentation fault      (core dumped) python3 manage.py migrate
```

在arm64上用1.7版本的镜像可以运行，避免以上错误。但是上传文件后不能成功处理文件。

## 自动备份

### 服务器端

``` console
mkdir ~/backup
mkdir ~/log
mkdir -p ~/tool/bin
```

创建脚本~/tool/bin/backup/backupPaperless-ngx.sh
内容如下

```bash
echo `date` start backup paperless-ngx
cd ~/docker/paperless-ngx
docker-compose exec -T webserver document_exporter ../export -z
ls -l ~/docker/paperless-ngx/export
mv ~/docker/paperless-ngx/export/*.zip ~/backup
echo `date` end backup paperless-ngx
```

``` console
chmod +x ~/tool/bin/backup/backupPaperless-ngx.sh
```

创建crontab任务

``` console
crontab -e
```

增加如下行

``` crontab
0 1 * * 1 /home/ubuntu/tool/bin/backupPaperless-ngx.sh >> /home/ubuntu/log/backupPaperless-ngx.log
```

服务器syncthing

``` yml
    syncthing:
        image: lscr.io/linuxserver/syncthing:latest
        environment:
            - PUID=1001
            - PGID=1001
            - TZ=Asia/Shanghai
        volumes:
            - ~/caddy-v2ray-docker/syncthing/config:/config
            - ~/caddy-v2ray-docker/caddy/srv:/caddy_srv
            - ~/backup:/backup
        ports:
            - 8384:8384
            - 22000:22000/tcp
            - 22000:22000/udp
            - 21027:21027/udp
        restart: unless-stopped
```

访问syncthing,增加目录/backup.

### 客户端

``` console
mkdir -p ~/docker/syncthing/config
```

~/docker/syncthing/docker-compose.yml文件内容如下

``` yml
version: "2.1"
services:
  syncthing:
    image: lscr.io/linuxserver/syncthing:latest
    container_name: syncthing
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
    volumes:
      - ~/docker/syncthing/config:/config
      - /media/nas001/02_backup/backup_aaray02:/backup
    ports:
      - 8384:8384
      - 22000:22000/tcp
      - 22000:22000/udp
      - 21027:21027/udp
    restart: unless-stopped
```

``` console
cd ~/docker/syncthing
docker-compose up -d
```

浏览器访问http://<server_ip>:8384

连接服务器和客户端的syncthing。

服务器每周1自动导出备份并移动文件到~/backup目录。syncthing自动同步到客户端。
