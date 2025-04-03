---
title: dashy和frp反向隧道
date: 2022-12-13 14:09:46
tags:
---

Docker安装dashy面板

参考
> https://blog.laoda.de/archives/docker-compose-install-dashy
> https://hub.docker.com/r/lissy93/dashy
> https://hub.docker.com/r/stefangenov/dashy

搭建环境的时候lissy93/dashy这个docker镜像有问题。所以改用stefangenov/dashy。
注:在我的armbian上无论是lissy93还是stefangenov的最新image都有问题。而这两个包在Oracle VPS(arm64)上都可以正常使用。

服务器192.168.1.100(192.168.1.108)
~/docker/dashy

docker-compose.yml

```yml
version: '3.3'
services:
    dashy:
        ports:
            - '80:80'
        volumes:
            - '~/docker/dashy/conf.yml:/app/public/conf.yml'
            - '~/docker/dashy/icons:/app/public/item-icons/icons'
        container_name: dashy
        restart: unless-stopped
        image: 'stefangenov/dashy'
```

icons目录来自https://github.com/walkxcode/dashboard-icons

instance启动之后会有错误。可以用docker exec -it da96 /bin/sh命令进到控制台，执行log里提示的命令。

cha256 哈希加密，地址用这个： https://emn178.github.io/online-tools/sha256.html

```yml
auth:
    users:
      - user: Roy
        hash: 6477fd513dcd82a78866654f9d064e8fea54a9d95e65c95ce6aee0699bf0948c
        type: admin
      - user: gugu
        hash: 8b2d38b789e90bb18567c2be4abbd4295f461f6453dd0447a3bf248a75eb0ae7
        type: normal
    enableGuestAccess: true
```

## 在04上启动dashy

端口4380

```console
ubuntu@instance4-arm64:~/docker/dashy_lissy93$ pwd
/home/ubuntu/docker/dashy_lissy93
ubuntu@instance4-arm64:~/docker/dashy_lissy93$ ls -l
total 20
-rw-rw-r-- 1 ubuntu ubuntu 9236 Mar 17 07:45 conf.yml
-rw-rw-r-- 1 ubuntu ubuntu  330 Mar 17 07:43 docker-compose.yml
drwxr-xr-x 5 ubuntu ubuntu 4096 Feb  7 04:33 icons
ubuntu@instance4-arm64:~/docker/dashy_lissy93$ cat docker-compose.yml
version: '3.3'
services:
    dashy:
        ports:
            - '4380:80'
        volumes:
            - '~/docker/dashy_lissy93/conf.yml:/app/public/conf.yml'
            - '~/docker/dashy_lissy93/icons:/app/public/item-icons/icons'
        container_name: dashy
        restart: unless-stopped
        image: 'lissy93/dashy'
ubuntu@instance4-arm64:~/docker/dashy_lissy93$ docker-compose up -d
```

## 打开防火墙

虚拟机之间10.0.0.0/24之间4000-4999防火墙打开

## 设置caddy并添加basicauth

```console
ubuntu@instance4-arm64:~/docker$ caddy hash-password
Enter password: 
Confirm password: 
$2a$14$kQv1kEuTt5UCdRJZ.JwaNe6DkzRdWJLD3xaRJ.F9AUo0s1LoWIy/.

ubuntu@instance4-arm64:~/docker/dashy_lissy93$ cat /etc/caddy/Caddyfile
<04>.theworkpc.com {
	respond "hello ray 04"
	handle_path /syncthing/* {
		reverse_proxy http://localhost:8384 {
			header_up Host {upstream_hostport}
		}
	}
}

dash.<ddns>.theworkpc.com {
	basicauth {
		<user> $2a$14$kQv1kEuTt5UCdRJZ.JwaNe6DkzRdWJLD3xaRJ.F9AUo0s1LoWIy/.
	}
	reverse_proxy 127.0.0.1:4380
}
```

## 重启caddy

```console
sudo systemctl restart caddy
```

## 设置反向FRP以映射到内网服务器

FRP默认是把本地的服务映射到在公网服务器上。
我的需求是把在外网服务器上运行的服务映射到本地，在局域网内访问。
本来这个需求可以通过ssh隧道实现。但是不知道是不是外网服务器在GFW的另一侧，ssh隧道可以映射，但是图片显示不出来。除了GFW想不出原因。

于是想借用之前已经建立的FRP作为隧道，在外网服务器运行frpc,在内网服务器运行frps，把外网的dashy(端口4380)映射到本地4380端口。最后通过nginx的反向代理映射到80端口。

### 原FRP隧道

服务器外网02,端口8888
客户端内网100

### 在内网100上运行frps

```frps.ini
[common]
bind_port = 7000
```

参考[vaultwarden 用docker搭建vaultwarden(bitwarden)服务器](/2023/01/25/vaultwarden)中关于frp的部分

创建/etc/systemd/system/frps.service文件
```service
[Unit]
Description=Frp Server Service
After=network.target syslog.target
Wants=network.target

[Service]
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/home/user/frp/frps -c /home/user/frp/frps.ini

[Install]
WantedBy=multi-user.target
```

enable并start frps

```console
sudo systemctl enable frps
sudo systemctl start frps
```

这一步完成在100上7000端口启动frps
下一步就是让02服务器借助原有的frp连接访问这个端口

### 修改100上的frpc.ini并重启frpc

在/home/user/frp/frpc.ini文件添加如下内容
```ini
[frps]
type = tcp
local_ip = 127.0.0.1
local_port = 7000
remote_port = 7000
```

重启
```console
sudo systemctl restart frpc
```

这一步就把100上的7000端口映射到原frp服务器端(02)上的7000端口。以后在02上访问本机7000端口就相当于访问100的7000端口。

### 在外网02上运行frpc

注: dashy是运行在04上的(10.0.0.127)

```/home/ubuntu/frp/frpc.ini
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
```

frpc连接的是本机的7000端口。把10.0.0.127(04)的4380端口映射到服务器(100)的4380端口。

创建/etc/systemd/system/frpc.service文件
```service
# 1. put frpc and frpc.ini under /usr/local/frpc/
# 2. put this file (frpc.service) at /etc/systemd/system
# 3. run `sudo systemctl daemon-reload && sudo systemctl enable frpc && sudo systemctl start frpc`
# Then we can manage frpc with `sudo service frpc {start|stop|restart|status}`
# See also: https://nosame.net/use-frp-to-reverse-proxy-your-nas/

# Alternative for server:
# - Offical: https://github.com/fatedier/frp/blob/a4cfab6/conf/systemd/frpc%40.service

[Unit]
Description=Frp Client Service
After=network.target
Wants=network.target network-online.target

[Service]
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/home/ubuntu/frp/frpc -c /home/ubuntu/frp/frpc.ini

[Install]
WantedBy=multi-user.target
```

enable并start frpc

```console
sudo systemctl enable frpc
sudo systemctl start frpc
```

启动后，在内网100的服务器上4380端口就映射了04上的4380端口。
此时，已经可以通过http://192.168.1.100:4380访问04服务器上的dashy了。

### 在内网100上设置nginx

```/etc/nginx/sites-available/myproxy
server {
	listen 11443 ssl;
	listen [::]:11443 ssl;
	include snippets/self-signed.conf;
	include snippets/ssl-params.conf;

	server_name _;

	location / {
		proxy_pass http://localhost:11080;
	}
}

server {
	listen 80 default_server;
	listen [::]:80 default_server;

        #listen 443 ssl;
        #listen [::]:443 ssl;
        #include snippets/self-signed.conf;
        #include snippets/ssl-params.conf;

        server_name _;

        location / {
                proxy_pass http://localhost:4380;
        }
}
```

第1个11443是https的运行在100上的vaultwarden
第2个80是运行在04上的dashy
