---
title: hexo-theme-Yin
date: 2020-05-14 16:41:00
tags:
---

一个简单的Hexo Theme. [hexo-theme-Yin](https://github.com/yin-hongwei/hexo-theme-yin)

编译的时候报了一些undefine的错误，只要在
themes/hexo-theme-Yin/_config.yml文件中加入以下内容即可

```text
gitalk: 
   enable: false
disqus:
   enable: false
reward:
   enable: false
```
