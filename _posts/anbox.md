---
title: anbox
date: 2020-07-30 14:29:05
tags:
---

![anbox](/myimages/anbox_1.png "anbox")

Anbox是Linux下的安桌“模拟器”。与其他模拟器不同，它不是虚拟机。性能较快。

[Anbox主页](https://anbox.io/)

如果没有声音，可以参考[这个帖子](https://github.com/anbox/anbox/issues/904)

基本思路是，需要拷几个文件到anbox下，但是anbox文件系统是只读的，所以anbox提供了“Android rootfs overlay”，可以在Linux下添加文件到anbox文件系统中。

[Android rootfs overlay](https://docs.anbox.io/userguide/advanced/rootfs_overlay.html)

需要添加的文件在[这里 https://github.com/anbox/anbox/tree/master/android/media](https://github.com/anbox/anbox/tree/master/android/media)

Anbox默认只能运行x86的程序，这个[教程 Anbox: How To Install Google Play Store And Enable ARM (libhoudini) Support, The Easy Way](https://www.linuxuprising.com/2018/07/anbox-how-to-install-google-play-store.html)提供了一个脚本，可以安装Google Play和libhoudini, libhoudini可以让anbox运行ARM的代码。
