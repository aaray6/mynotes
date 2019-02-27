---
title: Kali Linux应用
date: 2019-02-22 09:59:29
tags:
---

# Kali Linux破解WPA/WPA2 Wifi密码

> [Kali Linux使用Aircrack破解wifi密码(wpa/wpa2)](http://topspeedsnail.com/kali-linux-crack-wifi-wpa/)
> [How To Hack WPA/WPA2 Wi-Fi With Kali Linux & Aircrack-ng](http://lewiscomputerhowto.blogspot.com/2014/06/how-to-hack-wpawpa2-wi-fi-with-kali.html)
> [How To Crack WPA/WPA2 Wi-Fi Passwords Using Aircrack-Ng In Kali](https://www.sunnyhoi.com/how-to-crack-wpawpa2-wi-fi-passwords-using-aircrack-ng-in-kali/)

Notes:

* `airodump-ng -c <channel> --bssid <WIFI BSSID> -w <Work directory> wlan0mon`

`<Work directory>`不光写目录，也要写目录下的文件前缀。例如/root/work/wifi/my.这样就会生成/root/work/wifi/my-01.*文件

* 抓到包后用airmon-ng stop wlan0mon命令结束网卡监听模式

* 如果对方密码不在字典里，也足够复杂，那就不用指望能破解出来了，所以这个要看运气

# Kali Metasploit

[Metasploit](/2019/02/27/metasploit/)

# apt package

* netstat

```console
apt install net-tools
```