---
title: syncthing的一些备注
date: 2025-04-04 22:55:33
tags:
---

X201上syncthing地址http://localhost:8384
syncthing的目录创建之后不能修改，但是可以接受符号链接。
我建立了链接~/dev/mynotes/public->../blog_heroku_github/caddy_srv
这个目录下一定要建个.stfolder的目录，没有内容，只是标记，否则syncthing不认。

直接链接到public这个方法方便但有不好的地方。hexo编译后直接就发布上去了。
我还是改回老方法。~/dev/blog_heroku_github/caddy_srv是目录。需要发布的时候用命令

```console
cp -pR ~/dev/mynotes/public/* ~/dev/blog_heroku_github/caddy_srv/
hexo d
```
