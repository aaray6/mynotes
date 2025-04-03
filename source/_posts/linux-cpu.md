---
title: linux cpu 相关工具
date: 2021-10-15 08:17:25
tags:
---

Ubuntu下根据文件查包的命令

```console
$ dpkg -S /usr/bin/indicator-cpufreq
indicator-cpufreq: /usr/bin/indicator-cpufreq
```

列出包下所有文件的命令

```console
$ dpkg -L indicator-cpufreq
/.
/usr
/usr/lib
/usr/lib/python3
/usr/lib/python3/dist-packages
/usr/lib/python3/dist-packages/indicator_cpufreq
/usr/lib/python3/dist-packages/indicator_cpufreq/__init__.py
/usr/lib/python3/dist-packages/indicator_cpufreq/indicator.py
...
```

根据这个帖子，可以安装cpupower工具

> https://wiki.archlinux.org/title/CPU_frequency_scaling_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)

```console
$ cpupower

Command 'cpupower' not found, but can be installed with:

sudo apt install linux-tools-common

$ sudo apt install linux-tools-common
...

$ cpupower frequency-info
WARNING: cpupower not found for kernel 4.15.0-159

  You may need to install the following packages for this specific kernel:
    linux-tools-4.15.0-159-generic
    linux-cloud-tools-4.15.0-159-generic

  You may also want to install one of the following packages to keep up to date:
    linux-tools-generic
    linux-cloud-tools-generic


$ sudo apt install linux-tools-4.15.0-159-generic linux-cloud-tools-4.15.0-159-generic linux-tools-generic linux-cloud-tools-generic
...

$ cpupower frequency-info
analyzing CPU 0:
  driver: acpi-cpufreq
  CPUs which run at the same hardware frequency: 0
  CPUs which need to have their frequency coordinated by software: 0
  maximum transition latency: 10.0 us
  hardware limits: 1.20 GHz - 2.67 GHz
  available frequency steps:  2.67 GHz, 2.67 GHz, 2.53 GHz, 2.40 GHz, 2.27 GHz, 2.13 GHz, 2.00 GHz, 1.87 GHz, 1.73 GHz, 1.60 GHz, 1.47 GHz, 1.33 GHz, 1.20 GHz
  available cpufreq governors: conservative ondemand userspace powersave performance schedutil
  current policy: frequency should be within 1.20 GHz and 2.67 GHz.
                  The governor "powersave" may decide which speed to use
                  within this range.
  current CPU frequency: Unable to call hardware
  current CPU frequency: 1.27 GHz (asserted by call to kernel)
  boost state support:
    Supported: yes
    Active: yes
    33999 MHz max turbo 4 active cores
    33999 MHz max turbo 3 active cores
    33999 MHz max turbo 2 active cores
    33999 MHz max turbo 1 active cores

```

设置CPU以最低频率powersave运行

```console
sudo cpupower frequency-set -g powersave
```

设置CPU最大频率为2GHz

```console
sudo cpupower -c all frequency-set --max 2GHz
```

看CPU信息

```console
sudo i7z
```

CPU 100%负载

> https://www.bgegao.com/2019/05/1227.html

```console
for i in `seq 1 $(cat /proc/cpuinfo |grep "physical id" |wc -l)`; do dd if=/dev/zero of=/dev/null & done
```

上述程序的结束可以使用：

1. fg 后按 ctrl + C (因为该命令是放在后台执行)

2. pkill -9 dd

永久设置启动时powersave

> https://sleeplessbeastie.eu/2015/11/09/how-to-set-cpu-governor-at-boot/

仿照上面方法，开机时设置CPU最大频率2GHz

```console
cat << EOF | sudo tee /etc/systemd/system/cpupower.service
[Unit]
Description=CPU max 2G
[Service]
Type=oneshot
ExecStart=/usr/bin/cpupower -c all frequency-set --max 2GHz
[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload

sudo systemctl enable cpupower.service
```
