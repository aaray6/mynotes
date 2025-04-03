---
title: 用 Hexo和Heroku搭建个人博客分别部署到github.io, heroku和VPS自建Caddy服务器上
date: 2022-05-26 18:08:38
tags:
---

同一个hexo目录，分别部署到

1. github.io
2. heroku
3. VPS自建Caddy服务器上

## 部署到github.io上

在_config.yml中配置了

```yml
deploy:
  type: git
  repo: git@github.com:aaray6/aaray6.github.io.git
  branch: master
```

用如下命令可以自动部署到aaray6.github.io中

```console
cd ~/dev/blog_heroku_github
hexo  g
hexo d
```

[github主页](https://aaray6.github.io)

## 部署到heroku上

参考:
[Build a Blog with Hexo and Heroku](https://medium.com/@hannahroach_58244/how-to-build-a-blog-with-hexo-heroku-and-graphcomment-c050dffe5d1a)

在项目根目录下执行了以下命令把整个目录放到了git@github.com:aaray6/aaray2.git上

```console
cd ~/dev/blog_heroku_github
git init
git remote add origin git@github.com:aaray6/aaray2.git
git push -u origin master
git add .
git commit -m "Add new files"
git push
```

在code里提交后，因为heroku的aaray2和github的repo aaray2建立了链接和自动部署，变更提交到github之后会自动发布到heroku上

[Heroku主页](https://aaray2.herokuapp.com/)

20221213: log里报如下错误

```log
2022-12-13T10:52:18.138418+00:00 heroku[router]: at=error code=H14 desc="No web processes running" method=GET path="/" host=aaray2.herokuapp.com request_id=ded365a1-c14c-4b86-97c3-b8818edb4753 fwd="152.70.249.162" dyno= connect= service= status=503 bytes= protocol=https
```

根据https://stackoverflow.com/questions/41804507/h14-error-in-heroku-no-web-processes-running里说的方法，运行

```console
$ heroku ps:scale web=1 --app aaray2
Scaling dynos... !
 ▸    Subscribe to Eco to scale your dynos. Learn more at https://blog.heroku.com/new-low-cost-plans
```

注: 从2022-11开始heroku没有免费的dyno,所以才出现以上异常。想解决必须花钱订阅。

## Caddy服务器

1. 用Docker部署Caddy服务器

2. 本地用如下命令生成静态网站后，打包上传到服务器的~/caddy-x2xxx-docker/caddy/srv/目录下

``` console
cd ~/dev/blog_heroku_github
hexo g
tar czvf public.tgz public/
```

upload public.tgz

在服务器上解压并拷贝覆盖老文件

```console
tar xzvf public.tgz
cp -R public/* ~/caddy-x2xxx-docker/caddy/srv/
```

[VPS主页](https://aaray02.theworkpc.com/)
