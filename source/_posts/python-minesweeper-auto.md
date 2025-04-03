---
title: 用python自动扫雷
date: 2020-06-03 21:36:21
tags:
---

用Python写程序实现Windows下自动扫雷

原始代码[GoMine](https://github.com/yuhaibao324/GoMine)和帖子[自动扫雷 python](https://www.cnblogs.com/chestnut-egg/p/9302238.html)在这里

Fork了一份到我的github中[我的GoMine副本](https://github.com/aaray6/GoMine)

原文中提供的游戏下载链接失效了，我在这里找到的下载[Arbiter_0.52.3.zip](http://saolei.wang/Download/Arbiter_0.52.3.zip)。

或者到这个网盘下载

链接: https://pan.baidu.com/s/1lSFNQocGyscuiS4LyxDgIA  密码: 7u9a

## Windows下环境准备

我的系统是Win7 x64

### 下载Arbiter_0.52.3.zip解压运行

### 安装Python3

### 安装Git

### 下载代码

git clone git@github.com:aaray6/GoMine.git

如果出现

```console
git clone git@github.com:aaray6/GoMine.git
Cloning into 'GoMine'...
The authenticity of host 'github.com (13.250.177.223)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com,13.250.177.223' (RSA) to the list of know
n hosts.
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

那可能是因为我的系统里装了360安全卫士，阻止程序修改**C:\Windows\System32\drivers\etc\hosts**文件，那手工修改这个文件，加入以下行

```text
13.250.177.223	github.com
```

如果加完了出现提示

```console
git clone git@github.com:aaray6/GoMine.git
Cloning into 'GoMine'...
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

那需要生成ssh key并把public key加到github中

生成ssh key pair的命令，可以打开一个Git Bash(安装Git后菜单里可以找到)

```console
ssh-keygen -t rsa -b 4096 -C "<youname>@<youremailbox>"
```

生成的public key在这个目录中
/c/Users/Administrator/.ssh

详细步骤看这里[Connecting to GitHub with SSH](https://help.github.com/en/articles/connecting-to-github-with-ssh)

### 运行并用pip安装缺少的python模块

运行代码

```console
python Sl/GoMine.py
```

提示缺少模块，用以下命令安装pywin32和pillow

```console
pip install pywin32
pip install pillow
```

### 安装MS VS Code或者其他python GUI

## 运行测试程序

先运行扫雷
![Minesweeper Arbiter](/myimages/python-minesweeper-01.png)

再运行python Sl/GoMine.py

代码中第30行原来为
top += 101
我测试时修改为
top += 100
才可以正常运行

```dos
>python Sl/GoMine.py
找到窗口
窗口坐标：
0 158 0 271
插旗
2
2
插旗
2
2
插旗
2
2
插旗
3
3
插旗
3
...
```

## 修改程序以便能用于Win7自带的扫雷

TODO
