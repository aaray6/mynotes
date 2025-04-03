---
title: caddy(v1) basicauth proxy dashy
date: 2023-02-09 23:50:05
tags:
---

dashy在内网服务器108上。用docker启动dashy并监听80端口。
想在公网访问，并且加上密码验证。

公网服务器已经有Caddy v1运行。

两个服务器已经安装了frp。

在dynu.com上申请了动态域名并开了通配。比如我的域名是mypc.theworkpc.com指向公网服务器IP，dashy.mypc.theworkpc.com也自动解析为相同IP。

## 设置frpc

frpc安装在108的~/frpc目录
在~/frpc/frpc.ini文件中添加如下内容

``` ini
[dashy]
type = tcp
local_ip = 127.0.0.1
local_port = 80
remote_port = 12080
```

重启frpc(之前已经设置好frpc的service)

```console
sudo systemctl restart frpc
```

到公网服务器上用netstat命令检查12080端口是否已经监听。

## 设置Caddy v1

在Caddy中增加如下内容

``` Caddyfile
dashy.mypc.theworkpc.com {
  basicauth / <user> <password>
  proxy / 10.0.0.32:12080
}
```

注意，因为Caddy是通过docker运行的。所以不能写localhost。必须写服务器网卡IP 10.0.0.32。这个IP可以用ip -a命令查询。

重启caddy的container

``` console
$ docker-compose restart caddy
Restarting caddy-v2ray-docker_caddy_1 ... done
```

caddy是docker-compose.yml中caddy service的名字。

以上都完成之后就可以通过dashy.mypc.theworkpc.com来访问了。