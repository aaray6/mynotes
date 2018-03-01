---
title: ubuntu 16.04 notes
date: 2018-03-01 09:15:57
tags:
---
# Tips for Ubuntu 16.04

## Browser to support Java

Install Firefox 51. The new Firefox and Chrome don't support Java anymore.
Install IcedTea Java Web Start to support Java plugin.

## Full screen a window

```console
wmctrl -r "Cloud Desktop GCG - Citrix Receiver for Java" -b toggle,fullscreen
```

## Switch between multi-desktop

CTRL+ALT+ UP/DOWN/LEFT/RIGHT
SUPER + S

## check CPU Temp

```console
sudo apt install lm-sensors
...
quxr@quxr-ThinkPad-X201:~$ sensors
coretemp-isa-0000
Adapter: ISA adapter
Core 0:       +60.0°C  (high = +95.0°C, crit = +105.0°C)
Core 2:       +58.0°C  (high = +95.0°C, crit = +105.0°C)

acpitz-virtual-0
Adapter: Virtual device
temp1:        +65.0°C  (crit = +100.0°C)

thinkpad-isa-0000
Adapter: ISA adapter
fan1:        3464 RPM
temp1:        +65.0°C  
temp2:         +0.0°C  
temp3:         +0.0°C  
temp4:         +0.0°C  
temp5:         +0.0°C  
temp6:         +0.0°C  
temp7:         +0.0°C  
temp8:         +0.0°C  
```
