---
title: gitnote
date: 2023-02-14 21:14:02
tags:
---

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/14/gitnote_main-1676376689217.png)
之前的笔记基本都是文字，很少截图。原因就是因为截图太麻烦。
现在都用视频了，我连图片都不用，确实不方便。很多笔记回头自己看都不方便。
一张图可以代替很多文字。

直到我发现有gitnote这个软件。

gitnote我用到的一些功能
1. 可以像word一样编辑markdown文档
2. 可以很容易的插入图片。插入的图片可以自动上传到github并在md中插入github链接
3. 文件自动同步到github中，也可以用其他例如gitee等。支持版本控制。
4. 我可以将gitnote中的md内容直接复制到hexo中。

gitnote还有很多强大的功能，以后慢慢摸索。作者在bilibili上有视频教程。搜gitnote可以找到。

gitnote使用思路

## 在github上创建私有repo存放gitnote

国内访问首先需要代理。在linux下设置代理或者设置环境变量。
```console
env|grep PROXY
HTTP_PROXY=http://127.0.0.1:8118/
FTP_PROXY=http://127.0.0.1:8118/
ALL_PROXY=socks://127.0.0.1:1089/
NO_PROXY=localhost,127.0.0.0/8,::1
HTTPS_PROXY=http://127.0.0.1:8118/
```

127.0.0.1:8118是privoxy代理的端口，连接科学上网。

/etc/privoxy/config

```config
forward-socks5 / 127.0.0.1:1089 .
```

1089是科学上网的SOCKS端口。privoxy可以直接用科学软件代替。直接开放SOCKS和HTTP代理。

```json
  "inbounds": [
    {
      "tag": "proxy",
      "port": 1080,
      "listen": "0.0.0.0",
      "protocol": "socks",
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls"
        ]
      },
      "settings": {
        "auth": "noauth",
        "udp": true,
        "ip": null,
        "address": null,
        "clients": null
      },
      "streamSettings": null
    },
    {
      "tag": "http",
      "port": 1088,
      "listen": "0.0.0.0",
      "protocol": "http",
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls"
        ]
      },
      "settings": {
        "auth": "noauth",
        "udp": false
      }
    }
```

如果已经有repo，可以用以下方法clone到本地。

```console
env|grep PROXY
HTTP_PROXY=http://127.0.0.1:8118/
FTP_PROXY=http://127.0.0.1:8118/
ALL_PROXY=socks://127.0.0.1:1089/
NO_PROXY=localhost,127.0.0.0/8,::1
HTTPS_PROXY=http://127.0.0.1:8118/
cd ~/dev
git clone git@github.com:aaray6/mygitnote.git mygitnote
```

## 运行gitnote,选择添加本地已经GIT仓库

选择刚才clone的目录
![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/14/gitnote_open_local_folder-1676378737284.png)

通常使用方法还是比较直观，一看就明白。

困扰我的有以下几点。

### 插件

目前用到的是图床插件，有github和local
github是把图片上传到github上。需要提前申请一个public的repo。local则是把图片存到笔记目录下的.local目录中，各有优点。因为我需要在hexo中也使用这些图片，所以github是目前更好的选择。

![plugins](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/14/gitnote_plugins-1676379322400.png)

1. github插件设置

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/14/gitnote_plugin_github-1676379655872.png)

2. local插件不需要设置

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/14/gitnote_plugin_local-1676379713907.png)

## 插入图片

菜单上的图片插入并不好用，而是要从文件管理器中向gitnote中拖拽图片文件的方式。

![title](https://raw.githubusercontent.com/aaray6/mygitnote_images/main/gitnote/2023/02/14/gitnote_image_drag-1676379901592.png)

## 复制文章到hexo中

```console
cd ~/dev/blog_heroku_github
hexo new gitnote
INFO  Created: ~/dev/blog_heroku_github/source/_posts/gitnote.md
nano ~/dev/blog_heroku_github/source/_posts/gitnote.md
```

把这篇文章的内容整个复制到md文件中，然后正常编译发布

```console
hexo g
hexo d
```

通过syncthing发布

```console
cp -pR public/* caddy_srv
```
