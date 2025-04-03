---
title: HP Printer  DeskJet 1212
date: 2022-09-01 10:23:41
tags: 
---

让HP DeskJet 1212打印机在Win10, Win7, Ubuntu 18.04, Armbian (N1 20.04)下工作。

## Win10 + USB

最简单，官方提供Win10驱动。只要访问123.hp.com下载安装HP Smart软件就可以。
这个型号的打印机用的是hp-deskjet-1200-series驱动。

## Win7 + USB

有官方驱动，在这个网页中有链接。
https://www.freeprintersupport.com/hp-deskjet-1200-series-driver-download/

除了Win7+,官方还支持Mac OS驱动。

## Linux + USB

重点是Linux连接USB打印机并通过CUPS服务分享为网络打印机。
这样就可以将USB打印机变成网络打印机。除了支持电脑打印，还可以通过局域网，直接从手机和平板电脑上打印（安卓和苹果）。

基本步骤为
1. 安装CUPS服务。
2. 安装HP Linux驱动。

### X64 Ubuntu 18.04 (IBM X201笔记本)

####  系统自带CUPS服务。默认端口是631

netstat -na|grep 631命令可以看是否已经启动。
sudo systemctl status cups命令可以看服务是否存在以及状态。

如果没有安装，可以用以下命令安装
sudo apt update
sudo apt install cups

访问CUPS的URL是
http://127.0.0.1:631/
注: 用chrome浏览器访问CUPS有问题。需要用Firefox浏览器。

注：如果在CUPS管理界面中遇到权限问题，是因为我的帐号没有加入lpadmin组的原因。
用命令
sudo usermod -a -G lpadmin <user>
sudo usermod -a -G lp <user>
加入lpadmin就可以了。

#### 尽管有多个HP DeskJet 1200 series的Linux驱动，但是我只弄成功了[HPLIP 3.22.6](https://sourceforge.net/projects/hplip/files/hplip/)。

这个链接中提到的其他驱动我都没有成功。https://www.openprinting.org/printer/HP/HP-DeskJet_1200C

而且我必须用HPLIP带的hp-setup命令添加打印机才行，在CUPS的管理界面或者Linux系统设置中的打印机管理界面添加打印机，即使用了同样的设置和驱动，也无法成功驱动打印机。

安装方法是下载hplip-3.22.6.run,运行这个脚本自解压和安装。
过程中都按回车选择默认选项就行。

安装成功之后用hp-setup (带图形界面)或者hp-setup -i(字符界面)添加打印机。
添加的打印机会在CUPS中看到。

注: 
共享URL:http://192.168.1.6:631/printers/DeskJet_1212
Description:	DeskJet_1212
Location:	Home_LAN
Driver:	HP Deskjet 1200 Series, hpcups 3.22.6 (color)
Connection:	hp:/usb/DeskJet_1200_series?serial=CN26I422SB
Defaults:	job-sheets=none, none media=iso_a4_210x297mm sides=one-sided

Administration->Set Default Options可以修改Output Mode(Black Only Grayscale)
这里修改黑白打印。

### Armbian 20.04 (N1盒子)

最终目标是通过N1盒子把USB打印机HP DeskJet 1212变成网络打印机。

#### N1盒子装Armbian,参考以下这个链接

> [斐讯N1](/2021/10/08/N1/)

#### Armbian安装CUPS

```console
sudo apt update
sudo apt upgrade
sudo apt install cups
```

检查用户是否在lp和lpadmin组里

```console
groups
```

如果用户不在组里，把用户加进去

```console
sudo usermod -a -G lpadmin <user>
sudo usermod -a -G lp <user>
```

注: 由于我的系统长时间未更新，以及所用的镜像http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ 取消了http必须使用https，所以在suto apt update的时候失败。也无法更新certificate.

于是先禁用https验证

```console
touch /etc/apt/apt.conf.d/99verify-peer.conf \
&& echo >>/etc/apt/apt.conf.d/99verify-peer.conf "Acquire { https::Verify-Peer false }"
```

然后更新certificate

```console
sudo apt install ca-certificates
```

> [apt-get update failed because certificate verification failed because handshake failed on nodesource](https://askubuntu.com/questions/1095266/apt-get-update-failed-because-certificate-verification-failed-because-handshake)

更新证书之后就可以重新启用https验证了。方法是删除/etc/apt/apt.conf.d/99verify-peer.conf 文件

然后正常update, upgrade, install

注: CUPS默认只在127.0.0.1上监听631端口。由于N1没有本地浏览器，所以需要修改/etc/cups/cupsd.conf
原来是Listen 127.0.0.1:631
改为
Port 631
重启cups

```console
sudo systemctl stop cups
sudo systemctl start cups
```

然后就可以通过https://192.168.1.22:631/来访问N1上的CUPS了。

#### Armbian下的HPLIP驱动

hplip有现成的软件包，但是安装之后都无法使用。在N1, x86 lubuntu, x86 Q4OS上都试过，几个不同版本的都不能用。表现为用lsusb命令已经可以看到系统发现了usb打印机设备，但是运行hp-setup -i命令，选择usb之后，出现如下提示。

```console
error: No device selected/specified or that supports this functionality "hp-setup"
```

只有自己编译出来的hplip 3.22.6 (截止2022.8.31时最新版)能用hp-setup -i添加打印机。

跟x86/x64上不同，arm64下无法简单运行hplip-3.22.6.run成功编译驱动。因为驱动用到了prnt/hpcups/libImageProcessor-x86_32.so或libImageProcessor-x86_64.so文件。而这个文件没有arm64版本的。所以编译必须去掉对这个文件的依赖。

这个[链接](https://bugs.launchpad.net/hplip/+bug/1784989)中提到libImageProcessor-x86_64.so是"CDS feature". 3.18.6中没有对这个库的依赖，但是我尝试在Armbian上编译3.18.6,没有成功。python脚本多处报错。

以下是我用的方法。

I. 运行hplip-3.22.6.run，在自解压之后选择q退出。得到hplip-3.22.6目录

II. 进入hplip-3.22.6目录，运行install.py开始安装。在最后失败的时候，记录下最后几步的日志。

III. 重新运行./configure命令，添加--disable-imageProcessor-build参数

IV. 修改Makefile,删除所有跟ImageProcessor相关的内容，然后make clean, make, make install

> [移除代码对ImageProcessor依赖的patch](https://git.slackware.nl/current/tree/source/ap/hplip/0025-Remove-all-ImageProcessor-functionality-which-is-clo.patch)

我发现3.22.6代码中已经有设置可以通过参数去掉对ImageProcessor库的依赖，不需要手工删除。

> [linux patch 格式与说明](https://www.cnblogs.com/wuyuxin/p/7001320.html),这个帮我读懂patch文件

这个图是autoconf, automake各个文件的关系
![autoconf, automake各个文件的关系](/myimages/autoconf_automake.png "autoconf")

> [关于Makefile,Makefile.in,Makefile.am,Configure功能及相互关系的问题](https://www.cnblogs.com/lsgxeva/p/7592485.html)

TODO 找到更好的编译方法。

#### N1上的打印机

Description:	DeskJet_1212
Location:	N1_LAN
Driver:	HP Deskjet 1200 Series, hpcups 3.22.6 (color)
Connection:	hp:/usb/DeskJet_1200_series?serial=CN26I422SB
Defaults:	job-sheets=none, none media=iso_a4_210x297mm sides=one-sided

https://192.168.1.22:631/printers/DeskJet_1212

### Lubuntu 18.19 (EEEPC901 x86_32)

N1盒子的Armbian无法用WIFI,用EEEPC901的Lubuntu可以使用无线网卡，可以把打印机和EEEPC901放到任何地方。

#### CUPS

安装后修改配置文件/etc/cups/cupsd.conf如下，以便可以从远程访问
访问的URL是https://192.168.1.15:631/

```conf
LogLevel warn
PageLogFormat
MaxLogSize 0
# Allow remote access
Port 631
Listen /run/cups/cups.sock
Listen /var/run/cups/cups.sock
Browsing On
BrowseLocalProtocols dnssd
DefaultAuthType Basic
WebInterface Yes
<Location />
  # Allow remote access...
  Order allow,deny
  Allow all
</Location>
<Location /admin>
</Location>
<Location /admin/conf>
  AuthType Default
  Require user @SYSTEM
</Location>
<Location /admin/log>
  AuthType Default
  Require user @SYSTEM
  Order allow,deny
  Order allow,deny
  Allow @LOCAL
  Order allow,deny
  Allow @LOCAL
  Order allow,deny
  Allow @LOCAL
</Location>
<Policy default>
  JobPrivateAccess default
  JobPrivateValues default
  SubscriptionPrivateAccess default
  SubscriptionPrivateValues default
  <Limit Create-Job Print-Job Print-URI Validate-Job>
    Order deny,allow
  </Limit>
  <Limit Send-Document Send-URI Hold-Job Release-Job Restart-Job Purge-Jobs Set-Job-Attributes Create-Job-Subscription Renew-Subscription Cancel-Subscription Get-Notifications Reprocess-Job Cancel-Current-Job Suspend-Current-Job Resume-Job Cancel-My-Jobs Close-Job CUPS-Move-Job CUPS-Get-Document>
    Require user @OWNER @SYSTEM
    Order deny,allow
  </Limit>
  <Limit CUPS-Add-Modify-Printer CUPS-Delete-Printer CUPS-Add-Modify-Class CUPS-Delete-Class CUPS-Set-Default CUPS-Get-Devices>
    AuthType Default
    Require user @SYSTEM
    Order deny,allow
  </Limit>
  <Limit Pause-Printer Resume-Printer Enable-Printer Disable-Printer Pause-Printer-After-Current-Job Hold-New-Jobs Release-Held-New-Jobs Deactivate-Printer Activate-Printer Restart-Printer Shutdown-Printer Startup-Printer Promote-Job Schedule-Job-After Cancel-Jobs CUPS-Accept-Jobs CUPS-Reject-Jobs>
    AuthType Default
    Require user @SYSTEM
    Order deny,allow
  </Limit>
  <Limit Cancel-Job CUPS-Authenticate-Job>
    Require user @OWNER @SYSTEM
    Order deny,allow
  </Limit>
  <Limit All>
    Order deny,allow
  </Limit>
</Policy>
<Policy authenticated>
  JobPrivateAccess default
  JobPrivateValues default
  SubscriptionPrivateAccess default
  SubscriptionPrivateValues default
  <Limit Create-Job Print-Job Print-URI Validate-Job>
    AuthType Default
    Order deny,allow
  </Limit>
  <Limit Send-Document Send-URI Hold-Job Release-Job Restart-Job Purge-Jobs Set-Job-Attributes Create-Job-Subscription Renew-Subscription Cancel-Subscription Get-Notifications Reprocess-Job Cancel-Current-Job Suspend-Current-Job Resume-Job Cancel-My-Jobs Close-Job CUPS-Move-Job CUPS-Get-Document>
    AuthType Default
    Require user @OWNER @SYSTEM
    Order deny,allow
  </Limit>
  <Limit CUPS-Add-Modify-Printer CUPS-Delete-Printer CUPS-Add-Modify-Class CUPS-Delete-Class CUPS-Set-Default>
    AuthType Default
    Require user @SYSTEM
    Order deny,allow
  </Limit>
  <Limit Pause-Printer Resume-Printer Enable-Printer Disable-Printer Pause-Printer-After-Current-Job Hold-New-Jobs Release-Held-New-Jobs Deactivate-Printer Activate-Printer Restart-Printer Shutdown-Printer Startup-Printer Promote-Job Schedule-Job-After Cancel-Jobs CUPS-Accept-Jobs CUPS-Reject-Jobs>
    AuthType Default
    Require user @SYSTEM
    Order deny,allow
  </Limit>
  <Limit Cancel-Job CUPS-Authenticate-Job>
    AuthType Default
    Require user @OWNER @SYSTEM
    Order deny,allow
  </Limit>
  <Limit All>
    Order deny,allow
  </Limit>
</Policy>
JobPrivateAccess default
JobPrivateValues default
SubscriptionPrivateAccess default
SubscriptionPrivateValues default
```

修改之后重启cups服务

#### HPLIP_3.22.6驱动

驱动可以在x86_32的系统下正常编译，但是依赖包的安装过程会遇到问题，所以需要先手动安装依赖包。编译时选择定制安装，把所有的模块都选n。

I. 运行hplip-3.22.6.run然后选q退出，得到解压目录hplip-3.22.6

II. 运行以下命令安装依赖包

```console
sudo -H pip2 install opencv-python==4.2.0.32
sudo -H pip2 install scikit-image==0.14.1 
```

III. 运行install.py安装

```console
user@eeepc-901:~/Downloads/hplip/hplip-3.22.6$ ./install.py

HP Linux Imaging and Printing System (ver. 3.22.6)
HPLIP Installer ver. 5.1

Copyright (c) 2001-18 HP Development Company, LP
This software comes with ABSOLUTELY NO WARRANTY.
This is free software, and you are welcome to distribute it
under certain conditions. See COPYING file for more details.

Installer log saved in: hplip-install_Fri-02-Sep-2022_12:05:59.log

-
note: Defaults for each question are maked with a '*'. Press <enter> to accept the default.
 

INSTALLATION MODE
-----------------
Automatic mode will install the full HPLIP solution with the most common options.
Custom mode allows you to choose installation options to fit specific requirements.

Please choose the installation mode (a=automatic*, c=custom, q=quit) : c


INTRODUCTION
------------
This installer will install HPLIP version 3.22.6 on your computer.
Please close any running package management systems now (YaST, Adept, Synaptic, Up2date, etc).


DISTRO/OS CONFIRMATION
----------------------
Distro appears to be Ubuntu 18.10.

Is "Ubuntu 18.10" your correct distro/OS and version (y=yes*, n=no, q=quit) ? 


DRIVER OPTIONS
--------------
Would you like to install Custom Discrete Drivers or Class Drivers ( 'd'= Discrete Drivers*,'c'= Class Drivers,'q'= Quit)?   : 

Initializing. Please wait...


SELECT HPLIP OPTIONS
--------------------
You can select which HPLIP options to enable. Some options require extra dependencies.

Do you wish to enable 'Network/JetDirect I/O' (y=yes*, n=no, q=quit) ? n
Do you wish to enable 'Graphical User Interfaces (Qt4)' (y=yes*, n=no, q=quit) ? n
Do you wish to enable 'PC Send Fax support' (y=yes*, n=no, q=quit) ? n
Do you wish to enable 'Scanning support' (y=yes*, n=no, q=quit) ? n
Do you wish to enable 'HPLIP documentation (HTML)' (y=yes*, n=no, q=quit) ? n


ENTER USER PASSWORD
-------------------
Please enter the sudoer (quxr)'s password: 
 

INSTALLATION NOTES
------------------
Enable the universe/multiverse repositories. Also be sure you are using the Ubuntu "Main" Repositories. See: https://help.ubuntu.com/community/Repositories/Ubuntu for more information.  Disable the CD-ROM/DVD source if you do not have the Ubuntu installation media inserted in the drive.

Please read the installation notes. Press <enter> to continue or 'q' to quit: 


SECURITY PACKAGES
-----------------
AppArmor is installed. 
AppArmor protects the application from external intrusion attempts making the application secure

Would you like to have this installer install the hplip specific policy/profile (y=yes*, n=no, q=quit) ? 


RUNNING PRE-INSTALL COMMANDS
----------------------------
OK


RUNNING HPLIP LIBS REMOVE COMMANDS
----------------------------------
sudo apt-get remove libhpmud0 libsane-hpaio printer-driver-postscript-hp
sudo apt-get remove libhpmud0 libsane-hpaio printer-driver-postscript-hp ( hp_libs_remove step 1)
OK


MISSING DEPENDENCIES
--------------------
Following dependencies are not installed. HPLIP will not work if all REQUIRED dependencies are not installed and some of the HPLIP features will not work if OPTIONAL dependencies are not installed.
Package-Name         Component            Required/Optional   
Do you want to install these missing dependencies (y=yes*, n=no, q=quit) ? 


INSTALL MISSING REQUIRED DEPENDENCIES
-------------------------------------
note: Installation of dependencies requires an active internet connection.


RUNNING SCANJET DEPENDENCY COMMANDS
-----------------------------------
sudo apt-get install --assume-yes python-pip (Scanjet-depend step 1)
sudo pip2 install --upgrade pip (Scanjet-depend step 2)
sudo apt-get install --assume-yes libleptonica-dev (Scanjet-depend step 3)
sudo apt-get install --assume-yes tesseract-ocr (Scanjet-depend step 4)
sudo apt-get install --assume-yes libtesseract-dev (Scanjet-depend step 5)
sudo -H pip2 install tesserocr (Scanjet-depend step 6)
warning: Failed to install this Scanjet dependency package. Some Scanjet features will not work.
sudo apt-get install --assume-yes tesseract-ocr-all (Scanjet-depend step 7)
sudo apt-get install --assume-yes libzbar-dev (Scanjet-depend step 8)
sudo apt-get install --assume-yes python-zbar (Scanjet-depend step 9)
sudo -H pip2 install opencv-python (Scanjet-depend step 10)
sudo -H pip2 install PyPDF2 (Scanjet-depend step 11)
sudo -H pip2 install imutils (Scanjet-depend step 12)
sudo -H pip2 install pypdfocr (Scanjet-depend step 13)
sudo -H pip2 install scikit-image (Scanjet-depend step 14)
sudo -H pip2 install scipy (Scanjet-depend step 15)
OK


READY TO BUILD AND INSTALL
--------------------------
Ready to perform build and install. Press <enter> to continue or 'q' to quit: 


PRE-BUILD COMMANDS
------------------
OK


BUILD AND INSTALL
-----------------
Running './configure --with-hpppddir=/usr/share/ppd/HP --prefix=/usr --disable-qt4 --disable-qt5 --disable-doc-build --disable-cups-ppd-install --disable-foomatic-drv-install --disable-libusb01_build --disable-foomatic-ppd-install --disable-hpijs-install --disable-class-driver --disable-udev_sysfs_rules --disable-policykit --enable-cups-drv-install --enable-hpcups-install --disable-network-build --disable-dbus-build --disable-scan-build --disable-fax-build --enable-apparmor_build'
Please wait, this may take several minutes...
Command completed successfully.

Running 'make clean'
Please wait, this may take several minutes...
Command completed successfully.

Running 'make'
Please wait, this may take several minutes...
Command completed successfully.

Running 'sudo make install'
Please wait, this may take several minutes...
Command completed successfully.


Build complete.



POST-BUILD COMMANDS
-------------------
OK


HPLIP UPDATE NOTIFICATION
-------------------------
Do you want to check for HPLIP updates?. (y=yes*, n=no) : 


RESTART OR RE-PLUG IS REQUIRED
------------------------------
If you are installing a USB connected printer, and the printer was plugged in when you started this installer, you will need to either      
restart your PC or unplug and re-plug in your printer (USB cable only). If you choose to restart, run this command after restarting:        
hp-setup (Note: If you are using a parallel connection, you will have to restart your PC. If you are using network/wireless, you can ignore 
and continue).                                                                                                                              

Restart or re-plug in your printer (r=restart, p=re-plug in*, i=ignore/continue, q=quit) : 
Please unplug and re-plugin your printer now.  Press <enter> to continue or 'q' to quit: 


PRINTER SETUP
-------------
Would you like to setup a printer now (y=yes*, n=no, q=quit) ? 
Please make sure your printer is connected and powered on at this time.
Do you want to setup printer in GUI mode? (u=GUI mode*, i=Interactive mode) : i
Running 'hp-setup  -i' command....

HP Linux Imaging and Printing System (ver. 3.22.6)
Printer/Fax Setup Utility ver. 9.0

Copyright (c) 2001-18 HP Development Company, LP
This software comes with ABSOLUTELY NO WARRANTY.
This is free software, and you are welcome to distribute it
under certain conditions. See COPYING file for more details.

(Note: Defaults for each question are maked with a '*'. Press <enter> to accept the default.)


Using connection type: usb

 
Setting up device: hp:/usb/DeskJet_1200_series?serial=CN26I422SB



---------------------
| PRINT QUEUE SETUP |
---------------------


Please enter a name for this print queue (m=use model name:'DeskJet_1200'*, q=quit) ?DeskJet_1212
Using queue name: DeskJet_1212
Locating PPD file... Please wait.

Found PPD file: drv:///hp/hpcups.drv/hp-deskjet_1200_series.ppd
Description: 

Note: The model number may vary slightly from the actual model number on the device.

Does this PPD file appear to be the correct one (y=yes*, n=no, q=quit) ? 
Enter a location description for this printer (q=quit) ?EEEPC901_LAN
Enter additonal information or notes for this printer (q=quit) ?

Adding print queue to CUPS:
Device URI: hp:/usb/DeskJet_1200_series?serial=CN26I422SB
Queue name: DeskJet_1212
PPD file: drv:///hp/hpcups.drv/hp-deskjet_1200_series.ppd
Location: EEEPC901_LAN
Information: 


---------------------
| PRINTER TEST PAGE |
---------------------


Would you like to print a test page (y=yes*, n=no, q=quit) ? 

HP Linux Imaging and Printing System (ver. 3.22.6)
Testpage Print Utility ver. 6.0

Copyright (c) 2001-18 HP Development Company, LP
This software comes with ABSOLUTELY NO WARRANTY.
This is free software, and you are welcome to distribute it
under certain conditions. See COPYING file for more details.


HP Linux Imaging and Printing System (ver. 3.22.6)
System Tray Status Service ver. 2.0

Copyright (c) 2001-18 HP Development Company, LP
This software comes with ABSOLUTELY NO WARRANTY.
This is free software, and you are welcome to distribute it
under certain conditions. See COPYING file for more details.

warning: No display found.
error: hp-systray requires Qt4 GUI and DBus support. Exiting.
warning: Unable to connect to dbus. Is hp-systray running?
Printing test page to printer DeskJet_1212...
Test page has been sent to printer.

note: If an error occured, or the test page failed to print, refer to the HPLIP website
note: at: http://hplip.sourceforge.net for troubleshooting and support.


Done.

Done.


RE-STARTING HP_SYSTRAY
----------------------

HP Linux Imaging and Printing System (ver. 3.22.6)
System Tray Status Service ver. 2.0

Copyright (c) 2001-18 HP Development Company, LP
This software comes with ABSOLUTELY NO WARRANTY.
This is free software, and you are welcome to distribute it
under certain conditions. See COPYING file for more details.

warning: No display found.
error: hp-systray requires Qt4 GUI and DBus support. Exiting.
user@eeepc-901:~/Downloads/hplip/hplip-3.22.6$
```

#### 在CUPS中添加打印机

在CUPS中可以添加打印机，只要驱动选择HP Deskjet 1200 Series, hpcups 3.22.6 (color)(注意不是1200C)就可以正常。

### 路由器 + USB

USB打印机 (https://openwrt.org/docs/guide-user/services/print_server/p910ndprinterserver)

如果路由器有USB接口，可以连接打印机并通过路由器共享。

在openwrt下安装如下几个包，就可以增加共享打印机。
kmod-usb-printer
p910nd
luci-app-p910nd
luci-i18n-p910nd-zh-cn

安装之后会增加菜单“网络存储->打印服务器”
![打印服务器](/myimages/hp_printer_1212_openwrt1.png)

在Padavan下如果固件支持，可以在如下菜单中打开“启用 TCP/IP 原始端口”为“是(bidirectional)”
![启用 TCP/IP 原始端口](/myimages/hp_printer_1212_padavan1.png)

用这种方式通过路由器共享出来的打印机只能做到把USB口打印机变成网络接口打印机。客户端还是需要安装对应的打印机驱动程序才能使用。

在CUPS中，用如下命令安装打印机驱动

```console
sudo apt install hplip
```

如果出现问题，可以尝试

```console
sudo apt install hplip  --fix-missing
```

然后打开CUPS web控制台。

![cups Administration](/myimages/hp_printer_1212_cups1.png)
点击"Add Printer"按钮

![cups Administration](/myimages/hp_printer_1212_cups2.png)
选择AppSocket/HP JetDirect 并按Continue

![cups Administration](/myimages/hp_printer_1212_cups3.png)
Connection添“socket://192.168.1.101:9100”并按Continue
192.168.1.101是openwrt路由器IP地址，9100是在openwrt“打印服务器”中设置的“监听端口”

![cups Administration](/myimages/hp_printer_1212_cups4.png)
填写相关信息，并确定选中“Sharing: 	Share This Printer”按“Continue”

![cups Administration](/myimages/hp_printer_1212_cups5.png)
选HP并继续

![cups Administration](/myimages/hp_printer_1212_cups6.png)
选HP Deskjet 1200 Series, hpcups 3.21.2(en)并按“Add Printer”

![cups Administration](/myimages/hp_printer_1212_cups7.png)
按“Set Default Options”

![cups Administration](/myimages/hp_printer_1212_cups8.png)
添加成功，最终跳到这个界面

### 手机连接CUPS打印机

手机可以自动列出同局域网的CUPS打印机。如果没有，尝试安装如下包，重启cups

```console
sudo apt install avahi-daemon avahi-utils
sudo systemctl stop cups
sudo systemctl start cups
```
