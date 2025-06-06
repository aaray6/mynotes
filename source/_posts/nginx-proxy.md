---
title: nginx反向代理(非docker)
date: 2022-12-13 13:32:21
tags:
---

nginx反向代理
参考 https://www.reddit.com/r/portainer/comments/qek4zz/portainer_behind_an_external_nginx_proxy/

## 在X201上

代理了portainer和syncthing。并且设置默认root是webvirtmgr

/etc/nginx/sites-enabled目录下default是指向../sites-available/default的链接

```conf
##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# https://www.nginx.com/resources/wiki/start/
# https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
# https://wiki.debian.org/Nginx/DirectoryStructure
#
# In most cases, administrators will remove this file from sites-enabled/ and
# leave it as reference inside of sites-available where it will continue to be
# updated by the nginx packaging team.
#
# This file will automatically load configuration files provided by other
# applications, such as Drupal or Wordpress. These applications will be made
# available underneath a path with that package name, such as /drupal8.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##

# Default server configuration
#
upstream portainer {
    server 127.0.0.1:9443;
}

server {
	listen 80 default_server;
	listen [::]:80 default_server;

	# SSL configuration
	#
	#listen 443 ssl default_server;
	#listen [::]:443 ssl default_server;
	listen 443 ssl http2 default_server;
        listen [::]:443 ssl http2 default_server;

        server_name quxr-ThinkPad-X201;
        include snippets/self-signed.conf;
        include snippets/ssl-params.conf;
	#
	# Note: You should disable gzip for SSL traffic.
	# See: https://bugs.debian.org/773332
	#
	# Read up on ssl_ciphers to ensure a secure configuration.
	# See: https://bugs.debian.org/765782
	#
	# Self signed certs generated by the ssl-cert package
	# Don't use them in a production server!
	#
	# include snippets/snakeoil.conf;

	#portainer	
        location /portainer/ {
                proxy_pass https://portainer/;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "Upgrade";
                proxy_set_header Host $host;
	}

	#syncthing
        location /syncthing/ {
                proxy_pass http://127.0.0.1:8384/;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "Upgrade";
                proxy_set_header Host $host;
	}		

	# webvirtmgr
	location /static/ {
        	root /var/www/webvirtmgr/webvirtmgr; # or /srv instead of /var
        	expires max;
    	}
    	location / {
        	proxy_pass http://127.0.0.1:8001;
        	proxy_set_header X-Real-IP $remote_addr;
        	proxy_set_header X-Forwarded-for $proxy_add_x_forwarded_for;
        	proxy_set_header Host $host:$server_port;
        	proxy_set_header X-Forwarded-Proto $scheme;
        	proxy_connect_timeout 600;
        	proxy_read_timeout 600;
        	proxy_send_timeout 600;
        	client_max_body_size 1024M; # Set higher depending on your needs
    	}


	#default root
	#root /var/www/html;

	# Add index.php to the list if you are using PHP
	index index.html index.htm index.nginx-debian.html;

	server_name _;

	#location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
	#	try_files $uri $uri/ =404;
	#}

	# pass PHP scripts to FastCGI server
	#
	#location ~ \.php$ {
	#	include snippets/fastcgi-php.conf;
	#
	#	# With php-fpm (or other unix sockets):
	#	fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
	#	# With php-cgi (or other tcp sockets):
	#	fastcgi_pass 127.0.0.1:9000;
	#}

	# deny access to .htaccess files, if Apache's document root
	# concurs with nginx's one
	#
	#location ~ /\.ht {
	#	deny all;
	#}
}


# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
#server {
#	listen 80;
#	listen [::]:80;
#
#	server_name example.com;
#
#	root /var/www/example.com;
#	index index.html;
#
#	location / {
#		try_files $uri $uri/ =404;
#	}
#}
```

其中，private/public keys是用如下命令生成的

```console
sudo openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt
```

dhparam.pem是用如下命令生成的

```console
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```

## 在aaray01上

代理了syncthing

服务器的Nginx是装v2ray的时候装的，所以配置文件是/etc/nginx/conf/conf.d/v2ray.conf

```conf
server {
...
        # myConfig for syncthing
        location /syncthing/ {
                proxy_pass http://127.0.0.1:8384/;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "Upgrade";
                proxy_set_header Host localhost;
        }
}
    
server {
        listen 80;
        listen [::]:80;
        server_name aaray01.theworkpc.com;
        return 301 https://<mydomain>$request_uri;
}
```

"proxy_set_header Host localhost;"作用是让syncthing认为请求是从localhost来的。如果不设置，远程访问会出现"Host check error"

"return 301 https://<mydomain>$request_uri;"是让http请求自动跳转到https

## Caddy代理

没成功。需要注意的是，如果用docker运行Caddy,那localhost并不是host的localhost而是docker里的localhost。所以想连接到其他instance运行portainer，不能用localhost:9000。

## 在N1上Nginx

/etc/nginx/sites-enabled/myproxy

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

### 修改文件权限解决代理失败问题

文件权限问题。导致nginx的反向代理不好用。执行以下操作修复权限。

```console
# 检查nginx的运行id
$ ps aux | grep "nginx: worker process"
www-data    1658  0.0  0.3  61264  6296 ?        S    13:17   0:00 nginx: worker process
www-data    1659  0.0  0.3  61200  5652 ?        S    13:17   0:00 nginx: worker process
www-data    1660  0.0  0.3  61132  5380 ?        S    13:17   0:00 nginx: worker process
www-data    1661  0.0  0.3  61176  5616 ?        S    13:17   0:00 nginx: worker process
# 把权限改为www-data
$ cd /var/lib/nginx/
$ sudo chown -R www-data:root *
```