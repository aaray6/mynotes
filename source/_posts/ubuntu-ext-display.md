---
title: ubuntu18.04双显示器配置
date: 2019-02-27 13:57:57
tags:
---

环境Thinkpad x201, ubuntu18.04, 外接显示器1920x1080, VGA接口

> [how-to-add-display-resolution-for-an-lcd-in-ubuntu-12-04-xrandr-problem](https://askubuntu.com/questions/138408/how-to-add-display-resolution-for-an-lcd-in-ubuntu-12-04-xrandr-problem)

First, use cvt to create a new resolution mode.

```console
$ sudo cvt 1920 1080 60
[sudo] xxxx 的密码：
# 1920x1080 59.96 Hz (CVT 2.07M9) hsync: 67.16 kHz; pclk: 173.00 MHz
Modeline "1920x1080_60.00"  173.00  1920 2048 2248 2576  1080 1083 1088 1120 -hsync +vsync
```

Next, declare a new resolution mode.

```console
sudo xrandr --newmode "1920x1080_60.00" 173.00 1920 2048 2248 2576 1080 1083 1088 1120 -hsync +vsync
```

Next, find out the name of your video device.

```console
sudo xrandr -q
Screen 0: minimum 320 x 200, current 1280 x 1568, maximum 8192 x 8192
LVDS-1 connected primary 1280x800+0+768 (normal left inverted right x axis y axis) 261mm x 163mm
   1280x800      60.16*+  59.99    59.97    59.81    59.91    50.00  
   1280x720      60.00    59.99    59.86    59.74  
   1024x768      60.04    60.00  
   960x720       60.00  
   928x696       60.05  
   896x672       60.01  
   1024x576      59.95    59.96    59.90    59.82  
   960x600       59.93    60.00  
   960x540       59.96    59.99    59.63    59.82  
   800x600       60.00    60.32    56.25  
   840x525       60.01    59.88  
   864x486       59.92    59.57  
   800x512       60.17  
   700x525       59.98  
   800x450       59.95    59.82  
   640x512       60.02  
   720x450       59.89  
   700x450       59.96    59.88  
   640x480       60.00    59.94  
   720x405       59.51    58.99  
   684x384       59.88    59.85  
   680x384       59.80    59.96  
   640x400       59.88    59.98  
   576x432       60.06  
   640x360       59.86    59.83    59.84    59.32  
   512x384       60.00  
   512x288       60.00    59.92  
   480x270       59.63    59.82  
   400x300       60.32    56.34  
   432x243       59.92    59.57  
   320x240       60.05  
   360x202       59.51    59.13  
   320x180       59.84    59.32  
VGA-1 connected 1024x768+0+0 (normal left inverted right x axis y axis) 0mm x 0mm
   1024x768      60.00* 
   800x600       60.32    56.25  
   848x480       60.00  
   640x480       59.94  
HDMI-1 disconnected (normal left inverted right x axis y axis)
DP-1 disconnected (normal left inverted right x axis y axis)
  1920x1080_60.00 (0x1c8) 173.000MHz -HSync +VSync
        h: width  1920 start 2048 end 2248 total 2576 skew    0 clock  67.16KHz
        v: height 1080 start 1083 end 1088 total 1120           clock  59.96Hz
```

Mine was named "VGA-1". Finally, add your new resolution mode to the device/system.

```console
sudo xrandr --addmode VGA-1 1920x1080_60.00
quxr@quxr-ThinkPad-X201:~$ sudo xrandr -q
Screen 0: minimum 320 x 200, current 1280 x 1568, maximum 8192 x 8192
LVDS-1 connected primary 1280x800+0+768 (normal left inverted right x axis y axis) 261mm x 163mm
   1280x800      60.16*+  59.99    59.97    59.81    59.91    50.00  
   1280x720      60.00    59.99    59.86    59.74  
   1024x768      60.04    60.00  
   960x720       60.00  
   928x696       60.05  
   896x672       60.01  
   1024x576      59.95    59.96    59.90    59.82  
   960x600       59.93    60.00  
   960x540       59.96    59.99    59.63    59.82  
   800x600       60.00    60.32    56.25  
   840x525       60.01    59.88  
   864x486       59.92    59.57  
   800x512       60.17  
   700x525       59.98  
   800x450       59.95    59.82  
   640x512       60.02  
   720x450       59.89  
   700x450       59.96    59.88  
   640x480       60.00    59.94  
   720x405       59.51    58.99  
   684x384       59.88    59.85  
   680x384       59.80    59.96  
   640x400       59.88    59.98  
   576x432       60.06  
   640x360       59.86    59.83    59.84    59.32  
   512x384       60.00  
   512x288       60.00    59.92  
   480x270       59.63    59.82  
   400x300       60.32    56.34  
   432x243       59.92    59.57  
   320x240       60.05  
   360x202       59.51    59.13  
   320x180       59.84    59.32  
VGA-1 connected 1024x768+0+0 (normal left inverted right x axis y axis) 0mm x 0mm
   1024x768      60.00* 
   800x600       60.32    56.25  
   848x480       60.00  
   640x480       59.94  
   1920x1080_60.00  59.96  
HDMI-1 disconnected (normal left inverted right x axis y axis)
DP-1 disconnected (normal left inverted right x axis y axis)
```

setup in UI

![setup](/myimages/ubuntu-ext-display-setup.png)