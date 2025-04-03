---
title: debian12
date: 2023-08-28 09:17:46
tags:
---

原来的ubuntu18.04无法用vmware horizon client连接公司的电脑。升级到20.04和22.04后依然不行。只能重装。

ubuntu22.04装上可以，但是装了几个软件后，不是到是不是什么ssl相关库被替换了，又出现相同错误。
实在找不到解决方法，最后装debian12。之前试过mint，也连不上。

debian12安装后，装了timeshift软件给系统作了快照，以便出问题后可以回滚。

timeshift在debian12下工作正常。但在ubuntu22.04下，恢复之后Firefox等用snap安装的软件都无法正常使用。这也是我放弃ubuntu改用debian的原因。

debian安装之后，做了以下设置

1. 安装vmware horizon client
2. 安装chrome。下载chrome deb安装包，右键单击后选择“打开方式”用“软件安装”。前几次用sudo dpkg -i xxx.deb方式安装后，重启debian就无法进入桌面。不知道两者是否有关联。另外，chrome在菜单上不显示快捷方式。也无法固定到快捷栏。只能通过google-chrome命令启动。
3. 设置macos theme
4. 安装virtualbox
5. 设置trojan为user service。端口1080。脚本在~/.config/systemd/user/trojan.service
6. 安装onedrive
7. 安装Remmina。通过"软件"安装
8. 安装docker和docker-compose。用apt安装。方法参考官网。
9. 安装vscode。同上。
10. 安装notepadqq。同chrome，无法显示图标。