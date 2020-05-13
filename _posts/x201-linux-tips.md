---
title: IBM X201 Linux (ubuntu) 记录
date: 2019-03-09 13:55:09
tags:
---

[X201过热](#X201过热)； [指纹登录](#指纹登录)； [更换启动界面](#更换启动界面)； [修复分区表和GRUB引导](#不小心把分区表弄坏了); [休眠](#Ubuntu18-04休眠)

## xxd - Linux HEX editor

## [Linux Baidu Pan](https://segmentfault.com/a/1190000015719430)

## X201过热

安装以下软件，Linux比安装之前降温10度左右

lm_sensor (and afterwards I did run sensors-detect)
thermald
TLP
intel-ucode

### TLP

```console
sudo add-apt-repository ppa:linrunner/tlp
sudo apt-get update
sudo apt-get install tlp
## thinkpad
sudo apt-get install tp-smapi-dkms acpi-call-dkms
### uninstall
sudo apt-get remove tlp
sudo add-apt-repository --remove ppa:linrunner/tlp
##
/etc/default/tlp
##
sudo tlp start
sudo tlp-stat
```

### thermald

```console
sudo apt-get install thermald
```

### CPU freqs

```console
sudo apt-get install indicator-cpufreq
```

## 指纹登录

```console
## For Ubuntu 16.04 or greater:
sudo apt install libpam-fprintd fprint-demo
## For Ubuntu 15.04 or less
sudo add-apt-repository -y ppa:fingerprint/fprint
sudo apt-get update
sudo apt-get install libfprint0 fprint-demo libpam-fprintd gksu-polkit
## After that, you can test it by running fprint_demo and save the fingerprint with fprintd-enroll. This will automatically make your login screen require a finger swipe instead of a password.
```

## 更换启动界面

> [How to change boot splash screen in 18.04](https://askubuntu.com/questions/1046370/how-to-change-boot-splash-screen-in-18-04)
> [Plymouth Themes](https://www.gnome-look.org/browse/cat/108/)

to install:

1. copy file to /usr/share/plymouth/themes

2. run in terminal:

```console
sudo update-alternatives --install /usr/share/plymouth/themes/default.plymouth default.plymouth /usr/share/plymouth/themes/<your theme>/<your theme>.plymouth 100
sudo update-alternatives --config default.plymouth
sudo update-initramfs -u
```

> initramfs: initramfs is a tiny version of the OS that gets loaded by the bootloader, right after the kernel. It lives in RAM, and it provides *just* enough tools and instructions to tell the kernel how to set up the real filesystem, mount the HDD read/write, and start loading all the system services. It includes the stub of init, PID #1. If your initramfs is broken, your boot fails.
> update-initramfs is a script that updates initramfs to work with a new kernel. In the Debian universe, you shouldn't need to run this command manually except under very unusual circumstances - a post-install script automatically handles it for you when you install a new kernel package.

### 测试命令

```console
sudo apt install plymouth-x11

sudo plymouthd ; sudo plymouth --show-splash ; for ((I=0; I<10; I++)); do sleep 1 ; sudo plymouth --update=test$I ; done ; sudo plymouth --quit
```

## 不小心把分区表弄坏了

1. 先别慌，别往硬盘里写东西

2. 用晨风或类似WinPE工具盘启动电脑。用Partition Magic之类的软件提供的恢复丢失分区找回分区

3. 修复GRUB2.制作Ubuntu/Xubuntu/Lubuntu启动盘引导系统,用Boot Repair修复引导扇区，这个方法对双、多系统也有效，工具会识别出所有的可引导系统并加到Grub2菜单中。

> [Restore MBR from Ubuntu live CD / USB](https://www.howopensource.com/2011/08/restore-mbr-from-ubuntu-live-cd-usb/)

```console
sudo add-apt-repository ppa:yannubuntu/boot-repair
sudo apt-get update
sudo apt-get install -y boot-repair
boot-repair
```

## Ubuntu18.04休眠

注意**如果用固态硬盘，不应该使用休眠功能，否则影响固态硬盘寿命**

- 1 先确定交换分区至少跟物理内存一样大。

理论上交换分区小于物理内存也可以成功。而且最好比物理内存大一点。（我就是在扩展交换分区大小时把分区表弄坏的，花了半天的时间才恢复原样）

> [Add ‘Hibernate’ Option in Power Menu in Ubuntu 18.04](http://ubuntuhandbook.org/index.php/2018/05/add-hibernate-option-ubuntu-18-04/)
> [How can I hibernate on Ubuntu 16.04](https://askubuntu.com/questions/768136/how-can-i-hibernate-on-ubuntu-16-04)

- 2 测试系统支持休眠，注意，以下命令回立刻将系统休眠，执行前请确保已经保存你的工作。

```console
sudo systemctl hibernate
```

- 3 在菜单上增加休眠按钮

```console
sudo nano /etc/polkit-1/localauthority/50-local.d/com.ubuntu.enable-hibernate.pkla
```

It opens nano with an empty file. Copy the lines below and paste them into the nano window.

```text
[Re-enable hibernate by default in upower]
Identity=unix-user:*
Action=org.freedesktop.upower.hibernate
ResultActive=yes

[Re-enable hibernate by default in logind]
Identity=unix-user:*
Action=org.freedesktop.login1.hibernate;org.freedesktop.login1.handle-hibernate-key;org.freedesktop.login1;org.freedesktop.login1.hibernate-multiple-sessions;org.freedesktop.login1.hibernate-ignore-inhibit
ResultActive=yes
```

Restart your computer and click the link to install the gnome extension: **Hibernate Status Button**.(第一个参考帖子里有截图)

- 4 修改GRUB增加resume参数

找到交换分区:

```console
grep swap /etc/fstab
```

for me this returns (partial output)

```console
# swap was on /dev/sda3 during installation
```

/dev/sda3是安装系统时用的交换分区，如果安装之后没改过的话。

```console
sudoedit /etc/default/grub
```

To the line starting **GRUB_CMDLINE_LINUX_DEFAULT** add **resume=/dev/YourSwapPartition** to the section in quotes (replace with the the partition you identified earlier). Using my example:

```text
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash resume=/dev/sda3"
```

更新/etc/default/grub文件之后，运行以下命令更新grub以使之生效

```console
sudo update-grub
```

注： **除了像上面使用交换分区，还可以使用交换文件实现休眠功能**

### 在我重新创建了交换分区后，交换分区号和UUID都变了，于是需要用以下方法重新设置

注:**我曾经试过不用休眠功能，于是我把/etc/initramfs-tools/conf.d/resume文件删了，删掉后，Linux启动特别慢，内核启动大概10多秒，恢复这个文件并更新ramfs之后，变回3秒之内**

> [I have enabled hibernate but it isn't working. What can I do?](https://askubuntu.com/questions/196364/i-have-enabled-hibernate-but-it-isnt-working-what-can-i-do)

以下方法在Ubuntu18.04测试有效

There is two way to fix this.

- Editing the /etc/initramfs-tools/conf.d/resume file

    First get the UUID of the swap partition.

    ```console
     sudo blkid | grep swap
    ```

    This will output a line similar to this:

    ```console
    /dev/sda12: UUID="a14f3380-810e-49a7-b42e-72169e66c432" TYPE="swap"
    ```

    The actually line will not match with this. Copy the value of UUID in between "..." double quote.

    Open the resume file

    ```console
    sudo gedit /etc/initramfs-tools/conf.d/resume
    ```

    And in that file, add a line like this

    ```text
    RESUME=UUID=a14f3380-810e-49a7-b42e-72169e66c432
    ```

    Don't forget to replace the actual UUID value you get from step 1. Save the file and exit gedit

    Then in terminal, execute this command

    ```console
    sudo update-initramfs -u
    ```

You will now be able to resume from hibernation

- Editing the /etc/default/grub file.

    Open a terminal and execute the below command to open it

    ```console
    sudo gedit /etc/default/grub 
    ```

    There will be a line like GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
    . Edit the line to insert RESUME=UUID=<your-uuid-value-here\> after the word splash.

    For example in my case, the line looks like this after editing

    ```text
     GRUB_CMDLINE_LINUX_DEFAULT="quiet splash resume=UUID=a14f3380-810e-49a7-b42e-72169e66c432" 
    ```

    Make sure, you used your UUID value you get from sudo blkid | grep swap command.

    Then do this command

    ```console
     sudo update-grub
    ```

This also enable you to successfully get resumed from hibernate.

### 更改Login登录界面背景

> [How to Change the Ubuntu Login screen](https://vitux.com/how-to-change-login-lock-screen-background-in-ubuntu/)

```console
sudo gedit /usr/share/gnome-shell/theme/ubuntu.css
```

替换lockDialogGroup段如下

```css
#lockDialogGroup {
background: #2c001e url(file:///usr/share/backgrounds/gdmlock.jpg);
background-repeat: no-repeat;
background-size: cover;
background-position: center;
}
```

Alt-F2 -> r

### 电脑挂起后，禁用鼠标唤醒

>[鼠标移动唤醒计算机从挂起如何禁用这里操作？](https://www.helplib.com/ubuntu/article_157796)

```console
$ ls -l /lib/systemd/system-sleep
-rwxr-xr-x 1 root root  92 2月  22  2018 hdparm
-rwxr-xr-x 1 root root 212 3月  27 17:36 mouse_logitech
-rwxr-xr-x 1 root root 219 2月  21 21:58 unattended-upgrades

$ cat /lib/systemd/system-sleep/mouse_logitech
#!/bin/sh
if [! -r /sys/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.2/power/wakeup ]; then
 exit 0
fi
case"$1" in
 pre )
 echo disabled > /sys/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.2/power/wakeup
;;
esac
```

### 用[CloneZilla](https://clonezilla.org/)拷贝硬盘/分区

有linux的分区用Ghost复制后不好用，不知道是不是我用的版本太老了。用CloneZilla好用。下载iso，做启动盘，启动，复制