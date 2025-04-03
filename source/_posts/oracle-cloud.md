---
title: Oracle Cloud Caddy
date: 2023-02-01 12:50:50
tags:
---

## Arm 主机iptables

不知道为什么arm主机需要额外设置iptables才能允许访问80/443端口。否则Caddy无法获取证书，报"no route to host"错误。

``` console
  sudo iptables -I INPUT -p tcp -m tcp --dport 80 -j ACCEPT
  sudo iptables -I INPUT -p tcp -m tcp --dport 443 -j ACCEPT
  ```

## Caddy反向代理和load balance

### 安装Caddy v2

Caddy官网https://caddyserver.com/。在官网下载Linux arm64版本文件。上传到服务器的~/caddy/目录

```console
cd ~/caddy
ln -s caddy_linux_arm64 caddy
```

创建~/caddy/Caddyfile

``` yml
server.mydomain.com {
	reverse_proxy instancexxx.subnetyyy.vcnzzz.oraclevcn.com:4001 127.0.0.1:4001
	#respond "I am aaray03"

	handle_path /syncthing/* {
		reverse_proxy http://localhost:8384 {
			header_up Host {upstream_hostport}
		}
	}
}
```

### 启动Caddy

```console
cd ~/caddy
sudo ./caddy run
```

如果log里有ERROR关于证书错误，需要执行前面iptables命令打开80/443端口。

如果是后台启动，用
sudo ./caddy start命令

更新Caddyfile文件之后，用./caddy reload命令重新加载。

## 反向代理syncthing

https://docs.syncthing.net/users/reverseproxy.html

## load balance

Caddy不支持http和https节点混用。
1. 简单的http子节点

```yml
domainname {
	reverse_proxy host1:4001 127.0.0.1:4001

	handle_path /syncthing/* {
		reverse_proxy http://localhost:8384 {
			header_up Host {upstream_hostport}
		}
	}
}
```

2. 简单的https子节点

```yml
domainname {
	reverse_proxy https://<domainname2> {
		header_up Host {upstream_hostport}
	}

	handle_path /syncthing/* {
		reverse_proxy http://localhost:8384 {
			header_up Host {upstream_hostport}
		}
	}
}
```

## Caddy自动启动

```console
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy

cd ~/caddy
sudo ./caddy stop

sudo cp ~/caddy/Caddyfile /etc/caddy/Caddyfile

sudo systemctl restart caddy
```

配置文件是/etc/caddy/Caddyfile