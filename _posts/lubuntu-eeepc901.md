---
title: lubuntu_eeepc901
date: 2018-02-24 18:40:12
tags:
---
# My notes for installing lubuntu on eeepc901

## How to show GRUB memu if it skips

Hold "shift" key after BIOS load.

## How to edit GRUB parameter

Select a entry and press "e"
After edit the parameter, press Ctrl-X or F10 to boot

## lubuntu Linux can't display correctly after boot

First, edit GRUB paramter and delete parameter

```aconf
gfxmode $linux_gfx_mode
```

then start lubuntu Linux.
After start lubuntu Linux, open terminal LXTerminal

```console
$sudo vi /etc/default/grub
```

Uncomment line

```aconf
#GRUB_TERMINAL=console
```

```console
$sudo update-grub
```

This issue is about "eeepc 901 resolution can't be set 1024x600"

## VNC

[Configure VNC Server](https://www.server-world.info/en/note?os=Ubuntu_17.04&p=desktop&f=6)

```console
sudo apt -y install vnc4server
```

### set VNC password

```console
ubuntu@dlp:~$ vncpasswd 
Password:
Verify:
```

### start VNC server

```console
ubuntu@dlp:~$ vncserver :1 

New 'dlp:1 (ubuntu)' desktop is dlp:1

Creating default startup script /home/ubuntu/.vnc/xstartup
Starting applications specified in /home/ubuntu/.vnc/xstartup
Log file is /home/ubuntu/.vnc/dlp:1.log
```

### stop once

```console
ubuntu@dlp:~$ vncserver -kill :1 
Killing Xvnc4 process ID 2675
```

### modify ~/.vnc/xstartup and add following line to the end

```console
ubuntu@dlp:~$ vi ~/.vnc/xstartup

# add to the end
exec /usr/bin/mate-session &
```

### start with diplay number '1', screen resolution '800x600', color depth '24'

```console
ubuntu@dlp:~$ vncserver :1 -geometry 1024x600 -depth 16 
New 'ubuntu:1 (ubuntu)' desktop is ubuntu:1

Starting applications specified in /home/ubuntu/.vnc/xstartup
Log file is /home/ubuntu/.vnc/ubuntu:1.log
```