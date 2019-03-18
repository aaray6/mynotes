---
title: git notes
date: 2018-02-27 23:04:10
tags:
---
# git notes

## Adding an existing project to GitHub using the command line

> [Adding an existing project to GitHub using the command line](https://help.github.com/en/articles/adding-an-existing-project-to-github-using-the-command-line)

Note: **推荐使用SSH URL，可以不用每次提交都输入密码,需要客户端生成pub/pri key pair,然后通过Web控制台添加pub key到github/setting/SSH and GPG keys中**

```console
$ git init

$ git add .
# Adds the files in the local repository and stages them for commit. To unstage a file, use 'git reset HEAD YOUR-FILE'.

$ git commit -m "First commit"
# Commits the tracked changes and prepares them to be pushed to a remote repository. To remove this commit and modify the file, use 'git reset --soft HEAD~1' and commit and add the file again.

$ git remote add origin remote repository URL
# Sets the new remote
$ git remote -v
# Verifies the new remote URL

$ git push origin master
# Pushes the changes in your local repository up to the remote repository you specified as the origin
```

## Download/Clone remote github repository to local folder

```console
git clone REMOTE-URL LOCAL-FOLDER
```

for example: git clone git@github.com:aaray6/mytest.git mytest/

## to put existing content into repository (merge tips)

```console
git clone <URL> <somefolder>
cp -R <somefolder>/.git <existing folder>
```

## check remote URL

```console
git remote -v
```

## change remote URL (from https to ssh or from ssh to https)

get the URL
![img](/myimages/git_repository_url.png)

```console
git remote set-url origin <remote-URL>
```

## some commands

```console
git push *
```

## GIT CRLF

```console
git config core.autocrlf
git config --global core.autocrlf false
```

## Connecting to GitHub with SSH

> [Connecting to GitHub with SSH](https://help.github.com/en/articles/connecting-to-github-with-ssh)

## GIT with proxy

Linux推荐使用proxychains, Windows推荐使用proxifier(收费软件)

Linux下可以安装terminator终端，然后用以下命令启动terminator

```console
proxychains terminator
```

Windows下先运行proxifier,然后在git bash中的命令自动会呗proxifier使用代理

## 命令行从服务器同步最新的版本

在本地git仓库目录中

```console
/data/Download/backup/myinv$ git fetch origin
|DNS-request| github.com
|S-chain|-<>-127.0.0.1:2080-<><>-4.2.2.2:53-<><>-OK
|DNS-response| github.com is 13.250.177.223
|S-chain|-<>-127.0.0.1:2080-<><>-13.250.177.223:22-<><>-OK
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (1/1), done.
remote: Total 3 (delta 2), reused 3 (delta 2), pack-reused 0
展开对象中: 100% (3/3), 完成.
来自 github.com:aaray6/myinventory
   a69c9af..0447f8f  master     -> origin/master

/data/Download/backup/myinv$ git checkout
您的分支落后 'origin/master' 共 1 个提交，并且可以快进。
  （使用 "git pull" 来更新您的本地分支）

/data/Download/backup/myinv$ git pull
|DNS-request| github.com
|S-chain|-<>-127.0.0.1:2080-<><>-4.2.2.2:53-<><>-OK
|DNS-response| github.com is 13.229.188.59
|S-chain|-<>-127.0.0.1:2080-<><>-13.229.188.59:22-<><>-OK
更新 a69c9af..0447f8f
Fast-forward
 myinventory.trln | 31 +++++++++++++++++++++++++++++++
 1 file changed, 31 insertions(+)

/data/Download/backup/myinv$ git checkout
您的分支与上游分支 'origin/master' 一致。
```

注:**推荐使用Visual Studio Code可以简化操作**