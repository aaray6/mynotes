---
title: trojan-gfw
date: 2020-05-12 14:50:48
tags:
---

## Trojan 服务器端

### Create VPS (google) and open 80/443 ports

### 申请 domain name (xzy.ddnsfree.com) on dynu.com,配置动态域名解析

### Login VPS

### Install Trojan

 (Binary & Package Distributions) [https://github.com/trojan-gfw/trojan/wiki/Binary-&-Package-Distributions]

### Quickstart Script

```console
sudo bash -c "$(curl -fsSL https://raw.githubusercontent.com/trojan-gfw/trojan-quickstart/master/trojan-quickstart.sh)"
```

### Install自动获取证书工具

```console
sudo apt-get install wget
sudo wget https://dl.eff.org/certbot-auto -P /usr/local/bin
sudo chmod a+x /usr/local/bin/certbot-auto
sudo certbot-auto certonly --standalone -d xyz.ddnsfree.com --email yourname@yourdomain.com
```

```text
...
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/xyz.ddnsfree.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/xyz.ddnsfree.com/privkey.pem
   Your cert will expire on 2020-04-22. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot-auto
   again. To non-interactively renew *all* of your certificates, run
   "certbot-auto renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:
   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

> **/etc/letsencrypt/live/xyz.ddnsfree.com/fullchain.pem**
> **/etc/letsencrypt/live/xyz.ddnsfree.com/privkey.pem**
> **certbot-auto renew**

### 配置trojan服务器

```console
cd /usr/local/etc/trojan
vi config.json

...
    "password": [
        "yourpassword1",
        "yourpassword2"
    ],
...
    "ssl": {
        "cert": "/etc/letsencrypt/live/xyz.ddnsfree.com/fullchain.pem",
        "key": "/etc/letsencrypt/live/xyz.ddnsfree.com/privkey.pem",
...
```

### 运行trojan

sudo systemctl start trojan
sudo systemctl status trojan

## 客户端

### Linux

下载[Linux release 1.14.0](https://github.com/trojan-gfw/trojan/releases/tag/v1.14.0)
解压，修改配置文件(~/tool/trojan/config.json),运行(~/tool/trojan/trojan)

### Windows

下载[Windows release 1.14](https://github.com/trojan-gfw/trojan/releases/tag/v1.14.0)
解压，安装运行库，运行
