---
title: vaultwarden 用docker搭建vaultwarden(bitwarden)服务器
date: 2023-01-25 09:24:00
tags:
---

## 安装vaultwarden

```console
cd
mkdir -p docker/vaultwarden/data
docker pull vaultwarden/server:latest
docker run -d --name vaultwarden -v ~/docker/vaultwarden/data/:/data/ -p 11080:80 vaultwarden/server:latest
```

如果需要备份，~/docker/vaultwarden/data目录下有所有需要备份的内容。

### docker-compose.yml

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

```console
docker compose up -d
```

## 设置frp

### 服务器端

打开防火墙8888端口。

下载frp (https://github.com/fatedier/frp)

解压文件。修改frps.ini

```ini
[common]
bind_port = 8888
token = <password>
```

运行

```console
./frps -c frps.ini &
2023/01/24 16:11:38 [I] [root.go:206] frps uses config file: frps.ini
2023/01/24 16:11:38 [I] [service.go:200] frps tcp listen on 0.0.0.0:8888
2023/01/24 16:11:38 [I] [root.go:215] frps started successfully
```

### 服务器端systemd控制服务

> https://gofrp.org/docs/setup/systemd/

```console
sudo vim /etc/systemd/system/frps.service
```

文件内容如下

```ini
[Unit]
Description=Frp Server Service
After=network.target syslog.target
Wants=network.target

[Service]
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/home/ubuntu/frp/frps -c /home/ubuntu/frp/frps.ini

[Install]
WantedBy=multi-user.target
```

使用 systemd 命令，管理 frps

```console
# 启动frp
sudo systemctl start frps
# 停止frp
sudo systemctl stop frps
# 重启frp
sudo systemctl restart frps
# 查看frp状态
systemctl status frps
```

配置 frps 开机自启。

```console
sudo systemctl enable frps
```

### 客户端

下载frp (https://github.com/fatedier/frp)。因为客户端是armbian,所以需要下载arm64版本的。

解压文件。修改frpc.ini

```ini
[common]
server_addr = <server ip>
server_port = 8888
token = <password>

[vaultwarden]
type = tcp
local_ip = 127.0.0.1
local_port = 11080
remote_port = 11080
```

运行

```console
./frpc -c frpc.ini&
```

### 客户端systemd控制服务

> https://gist.github.com/imyelo/b6c3d3d9383f7d5623f06a0c11052530

```console
sudo vim /etc/systemd/system/frpc.service
```

文件内容如下

```ini
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
ExecStart=/home/user/frp/frpc -c /home/user/frp/frpc.ini

[Install]
WantedBy=multi-user.target
```

使用 systemd 命令，管理 frpc

```console
# 启动frp
sudo systemctl start frpc
# 停止frp
sudo systemctl stop frpc
# 重启frp
sudo systemctl restart frpc
# 查看frp状态
systemctl status frpc
```

配置 frps 开机自启。

```console
sudo systemctl enable frpc
```

## 绑定域名

noip.com申请免费域名并绑定服务器IP地址。

## Caddy映射

Caddy是跟Nginx类似的web服务器。用Caddy可以做到自动申请letsencrypt证书。我服务器的Caddy版本是v1版本，所以配置文件格式是老格式。

在Caddyfile文件根中增加如下一段

```yml
<绑定的域名> {
    proxy / http://10.0.0.32:11080 {
        transparent
    }
}
```

上面的10.0.0.32是服务器内网IP地址。11080是通过frp映射的端口号。

## 注册新帐号

浏览器打开https://域名访问vaultwarden。注册新帐号。

## 关闭注册帐号功能

停止并删除vaultwarden container，并用以下命令重新运行vaultwarden

```console
docker run -d --name vaultwarden -e SIGNUPS_ALLOWED=false -v ~/docker/vaultwarden/data/:/data/ -p 11080:80 vaultwarden/server:latest
```

## 局域网映射HTTPS访问

整个服务过于复杂，依赖

1. 本地服务器
2. 本地网络
3. 公网服务器
4. FPR服务器和客户端
5. Caddy

为防止服务不可用，设置在局域网访问。服务本身是HTTP协议11080端口。但是服务却不允许通过HTTP访问，必须通过HTTPS访问。
利用nginx设置反向代理通过HTTPS端口11443的步骤如下。

> https://www.techrepublic.com/article/how-to-enable-ssl-on-nginx/

1. 安装nginx

```console
sudo apt install nginx
```

2. 生成自签名证书

```console
sudo openssl req -x509 -nodes -days 9365 -newkey rsa:2048 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt
```

提示Common Name的时候输入IP地址

3. 配置nginx

```console
sudo nano /etc/nginx/snippets/self-signed.conf
```

文件内容

```conf
ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;
ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
```

然后，生成另外一个配置文件

```console
sudo nano /etc/nginx/snippets/ssl-params.conf
```

内容如下

```conf
ssl_protocols TLSv1.2;
ssl_prefer_server_ciphers on;
ssl_dhparam /etc/ssl/certs/dhparam.pem;
ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384;
ssl_ecdh_curve secp384r1; # Requires nginx >= 1.1.0
ssl_session_timeout 10m;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off; # Requires nginx >= 1.5.9
# ssl_stapling on; # Requires nginx >= 1.3.7
# ssl_stapling_verify on; # Requires nginx => 1.3.7
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
```

生成dhparam.pem文件

```console
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```

生成nginx配置文件myproxy

```console
sudo nano /etc/nginx/sites-available/myproxy
```

内容如下

```conf
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
```

替换配置文件,重启nginx

```console
cd /etc/nginx/sites-enabled/
sudo rm default 
sudo ln -s /etc/nginx/sites-available/myproxy myproxy
sudo systemctl restart nginx
```

## 参考

> https://www.hash070.top/archives/bitwarden-docker-deploy.html
> https://hub.docker.com/r/vaultwarden/server
