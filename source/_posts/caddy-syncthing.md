---
title: Docker Caddy反向代理syncthing
date: 2022-12-14 14:18:42
tags:
---

配置Caddy->syncthing的反向代理，并通过syncthing自动更新Caddy上的hexo网页。

Caddy是装代理的时候用docker装的。Caddy和代理在同一个docker-compose.yml里。

在docker-compose.yml里添加syncthing并映射到/syncthing/下。

## Docker Compose设置

### docker-compose.yml

中添加如下一段

PUID/PGID=1001是我运行docker的用户。不用root运行而用普通用户运行可以减少一些权限上的问题。
volumes中/home/ubuntu/caddy-v2ray-docker/caddy/srv:/caddy_srv是把Caddy的root目录(/home/ubuntu/caddy-v2ray-docker/caddy/srv)映射到容器中的/caddy_srv目录。
8384是syncthing的默认WebUI端口。

```yml
    syncthing:
        image: lscr.io/linuxserver/syncthing:latest
        #container_name: syncthing
        #hostname: aaray02.theworkpc.com #optional
        environment:
            - PUID=1001
            - PGID=1001
            - TZ=Asia/Shanghai
        volumes:
            - /home/ubuntu/caddy-v2ray-docker/syncthing/config:/config
            - /home/ubuntu/caddy-v2ray-docker/caddy/srv:/caddy_srv
        ports:
            - 8384:8384
            - 22000:22000/tcp
            - 22000:22000/udp
            - 21027:21027/udp
        restart: unless-stopped
```

### Caddyfile

caddy/Caddyfile文件。因为版本老，所以Caddy用的是v1的语法。如果是v2，语法完全不同。

```yml
<mydomainname> {
    log ./caddy.log
    root /srv
    gzip
    proxy /ws v2ray:8001 {
        websocket
        header_upstream -Origin
    }

    proxy /syncthing/ http://syncthing:8384 {
        #transparent
        without /syncthing/
    }

    timeouts {
        read none
        write none
        header none
    }
}
```

syncthing 文档https://docs.syncthing.net/users/reverseproxy.html#caddy设置如下但不好用

```yml
proxy /syncthing localhost:8384 {
    transparent
}

timeouts {
    read none
    write none
    header none
}
```

根据这个文档https://mdleom.com/blog/2020/05/23/caddy-upgrade-v2-proxy/设置成功。

```yml
proxy /img https://backend.com/img/blog {
  without /img
}
```

设置成

```yml
    proxy /syncthing/ http://syncthing:8384 {
        #transparent
        without /syncthing/
    }
```

proxy的第一个"/syncthing/"是要映射的目录，目录后面的/不能省略，否则会导致syncthing找不到相对目录下的js脚本。
第2个参数"http://syncthing:8384"前面的"http://"确保访问的是8483端口的http服务。syncthing是docker-compose.yml中syncthing这个容器的名字。同一个docker-compose.yml中的容器会自动加到同一个网络中，互相之间用名字可以解析到各个容器的IP地址。

### syncthing

1. 在我的X201和aaray02上的syncthing互相添加对方
2. 在X201上添加目录"caddy_srv"指向"~/dev/blog_heroku_github/caddy_srv"。因为hexo clean的时候会把整个public目录都删除，导致目录下的.stfolder目录也被删除。这个目录删除就会引起syncthing里目录出错。所以额外建了caddy_srv目录，每次更新后手动把public目录下的内容都拷贝到caddy_srv下。另外，在caddy_srv目录下rm -rf *也是安全的，不会删除.stfolder。
3. 共享这个目录给aaray02
4. 在aaray02上接受，并映射到"/caddy_srv"，名字是"caddy_srv"
5. 设置X201的"caddy_srv"目录为只上传，aaray02的"caddy_srv"为只接收