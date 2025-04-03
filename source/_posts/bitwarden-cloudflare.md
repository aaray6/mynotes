---
title: bitwarden相关及cloudflare加速
date: 2025-03-15 11:44:21
tags:
---

在家里运行了vaultwarden,通过公网服务器映射到caddy的一个网址。浏览器/IOS/Android分别装了插件和Bitwarden使用。以前一直好用，最近发现Android(鸿蒙)不能使用了。总是出现“我们无法处理您的请求 ”。而Web页面登陆没问题，苹果手机也可以。

解决问题过程记录以下内容供日后参考。

## Android客户端App不好用

跟鸿蒙无关。卸载后用APKPure重装2024.6月版本就好了。

## 更新服务器端

服务运行在100/108的N1 Armbian上。
~/docker/vaultwarden/docker-compose.yml

```yml
version: "2.1"
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    volumes:
      - ~/docker/vaultwarden/data/:/data/
    ports:
      - 11080:80
    restart: unless-stopped
```

用以下命令更新重启服务器端。但是更新后最新安卓客户端仍然报错。所以问题无法通过更新服务器端解决。

```console
docker-compose pull
docker-compose stop
docker-compose start
```

## frp

在公网服务器上运行frps,用N1的frpc连接到公网服务器frps,把vaultwarden映射到公网服务器上11080。

同时，在N1上运行frps服务器7000端口。映射到公网服务器的7000端口。在公网服务器上用frpc连接本机7000端口，就可以把服务器的服务映射到N1的对应端口4380(dashy), 4800(tm)。

公网服务器

frps.ini

```ini
[common]
bind_port = 8888
token = <password1>
```

frpc.ini

```ini
[common]
server_addr = 127.0.0.1
server_port = 7000

#[ssh]
#type = tcp
#local_ip = 127.0.0.1
#local_port = 22
#remote_port = 6000

[dashy]
type = tcp
local_ip = 10.0.0.127
local_port = 4380
remote_port = 4380

[tm]
type = tcp
local_ip = 10.0.0.155
local_port = 4800
remote_port = 4800

```

N1

frpc.ini

```ini
[common]
server_addr = <公网服务器IP>
server_port = 8888
token = <password1>

#[ssh]
#type = tcp
#local_ip = 127.0.0.1
#local_port = 22
#remote_port = 6000

[vaultwarden]
type = tcp
local_ip = 127.0.0.1
local_port = 11080
remote_port = 11080

#[dashy]
#type = tcp
#local_ip = 127.0.0.1
#local_port = 80
#remote_port = 12080

[frps]
type = tcp
local_ip = 127.0.0.1
local_port = 7000
remote_port = 7000

#[wordpress]
#type = tcp
#local_ip = 127.0.0.1
#local_port = 4880
#remote_port = 11443
```

frps.ini

```ini
[common]
bind_port = 7000
```

## Caddy

通过Caddy把11080端口映射到vw.我的二级免费域名上。

```json
vw.我的二级免费域名{
    proxy / http://10.0.0.32:11080 {
        transparent
    }
}
vw.我的eu域名.eu.org {
    proxy / http://10.0.0.32:11080 {
        transparent
    }
}
```

用以下命令单独重启docker-compose.yml中的某个服务

```console
docker-compose restart caddy
```

## cloudflare

在eu.org上注册免费域名，用cloudflare解析。

在cloudflare上添加CNAME记录

Type: CNAME
Name: vw
Content: vw.我的二级免费域名
Proxy status: Proxied
TTL: Auto

之后就可以通过vw.我的eu域名.eu.org来访问了。
