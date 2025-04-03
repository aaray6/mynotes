---
title: 用caddy通过redirect/rewrite提供tvbox接口
date: 2023-03-21 16:16:20
tags:
---

用caddy通过redirect/rewrite提供tvbox接口

想给老人的电视盒上装tvbox，但是因为tvbox的接口有时需要更换很不方便

于是考虑建一个网址，这个网址提供tvbox接口，这个接口只是一个跳板，最终返回的是别的接口的内容。

这样当接口失效的时候，我可以直接在服务器上重新映射接口，就可以对老人家的tvbox的接口进行更新。

服务器上已经安装了caddy,并申请了域名例如tvbox.domain.com

## 方法1:redirect (推荐)

在/etc/caddy/Caddyfile中添加如下内容

```json
tvbox.domain.com {
	redir /* https://yydsys.top/duo
}
```

##方法2:rewrite

在/etc/caddy/Caddyfile中添加如下内容

```json
tvbox.domain.com {
	handle / {
		rewrite * /duo
		reverse_proxy https://yydsys.top {
			header_up Host {upstream_hostport}
		}
	}
}
```

## 重新加载caddy配置

```console
#caddy reload
```

## 在tvbox的接口中设置为

https://tvbox.domain.com

## 几个好用的接口

https://yydsys.top/duo
http://饭太硬.ga/tv
https://agit.ai/Yoursmile7/TVBox/raw/branch/master/XC.json

> https://www.yydsys.top/box/api/

## curl测试方法

```cosole
curl --head https://tvbox.domain.com
HTTP/2 302 
alt-svc: h3=":443"; ma=2592000
location: https://yydsys.top/duo
server: Caddy
date: Tue, 21 Mar 2023 08:10:19 GMT
```

curl --head <url>是看request/response的头
302是redirect的作用
如果直接用curl <url>，返回的是网页内容。

## 参考网站

> https://mkyong.com/web/curl-display-request-headers-and-response-headers/
> https://caddy.community/t/example-for-redir/7475
> https://schacker.github.io/2018/07/27/rewrite%E4%B8%8Eredirect%E5%8C%BA%E5%88%AB/
> https://caddy.community/t/redirect-reverse-proxy-to-path/16173

