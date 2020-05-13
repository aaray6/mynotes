---
title: How to setup OpenVPN over ssr on gcloud
date: 2018-02-24 19:03:50
tags:
---

## apply VPC (google cloud compute engine)

First, you need to access Google :)

[Google Cloud](https://cloud.google.com)
[Google Cloud console](https://console.cloud.google.com/home)

Click `Compute Engine`
![gcloud console](/myimages/gcloud_console02.png)
Create new instance
![gcloud console](/myimages/gcloud_console03.png)
![gcloud console](/myimages/gcloud_console04.png)

## shadowsocks server

Refer to [this post](https://github.com/Alvin9999/new-pac/wiki/%E8%87%AA%E5%BB%BAss%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%95%99%E7%A8%8B)

## PC (install lubuntu on eeepc901)

Some notes about [lubuntu on eeepc901](lubuntu-eeepc901.md)

## shadowsocks client

Refer to [this post](https://github.com/Alvin9999/new-pac/wiki/%E8%87%AA%E5%BB%BAss%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%95%99%E7%A8%8B)

## server side openvpn

> [How To Set Up an OpenVPN Server on Debian 8](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-debian-8)
> [Setting up VPN using OpenVPN on Google Cloud or AWS](http://roshansingh.in/blog/2014/12/08/setting-up-vpn-using-openvpn-on-google-cloud-or-aws/)

Step 1~7

```console
sudo apt-get update
sudo apt-get install openvpn easy-rsa
sudo gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz > /etc/openvpn/server.conf
```

update server.conf as first doc step 2
*note: must use tcp proto because we need to use socks proxy*

```aconf
proto tcp
```

```console
sudo echo 1 > /proc/sys/net/ipv4/ip_forward
```

update /etc/sysctl.conf
...

### server F/W (ufw)

[How To Set Up an OpenVPN Server on Debian 8](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-debian-8) Step 4 — Install and Configure ufw

```console
apt-get install ufw
ufw allow ssh

myaccount@instance-1:/etc/openvpn$ sudo ufw status
Status: active
To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere
1194/udp                   ALLOW       Anywhere
443                        ALLOW       Anywhere
80                         ALLOW       Anywhere
22                         ALLOW       Anywhere (v6)
1194/udp                   ALLOW       Anywhere (v6)
443                        ALLOW       Anywhere (v6)
80                         ALLOW       Anywhere (v6)
myaccount@instance-1:/etc/openvpn$ sudo ufw allow 2295/tcp
Rule added
Rule added (v6)
myaccount@instance-1:/etc/openvpn$ sudo ufw allow 2295/udp
Rule added
Rule added (v6)
myaccount@instance-1:/etc/openvpn$ sudo ufw status
Status: active
To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere
1194/udp                   ALLOW       Anywhere
443                        ALLOW       Anywhere
80                         ALLOW       Anywhere
2295/tcp                   ALLOW       Anywhere
2295/udp                   ALLOW       Anywhere
22                         ALLOW       Anywhere (v6)
1194/udp                   ALLOW       Anywhere (v6)
443                        ALLOW       Anywhere (v6)
80                         ALLOW       Anywhere (v6)
2295/tcp                   ALLOW       Anywhere (v6)
2295/udp                   ALLOW       Anywhere (v6)
```

update `/etc/default/ufw`

```console
DEFAULT_FORWARD_POLICY="DROP"
```

This must be changed from DROP to ACCEPT. It should look like this when done:

```console
DEFAULT_FORWARD_POLICY="ACCEPT"
```

update `/etc/ufw/before.rules`

```console
#
# rules.before
#
# Rules that should be run before the ufw command line added rules. Custom
# rules should be added to one of these chains:
#   ufw-before-input
#   ufw-before-output
#   ufw-before-forward
#

# START OPENVPN RULES
# NAT table rules
*nat
:POSTROUTING ACCEPT [0:0]
# Allow traffic from OpenVPN client to eth0
-A POSTROUTING -s 10.8.0.0/8 -o eth0 -j MASQUERADE
COMMIT
# END OPENVPN RULES

# Don't delete these required lines, otherwise there will be errors
*filter
```

enable it

```console
ufw enable
```

### gcloud F/W (gcloud)

By default, gcloud only open 80 and 443 port. If we need to open more ports, can use following commands

```console
gcloud compute firewall-rules list
gcloud compute firewall-rules create my-allow-openvpn --allow tcp:2295,udp:2295 --description="openvpn"
```

![gcloud console](/myimages/gcloud_console01.png)
[gcloud compute firewall-rules](https://cloud.google.com/sdk/gcloud/reference/compute/firewall-rules/)

```console
gcloud compute firewall-rules create <rule-name> --allow tcp:9090 --source-tags=<list-of-your-instances-names> --source-ranges=0.0.0.0/0 --description="<your-description-here>"
```

delete rule

```console
gcloud compute firewall-rules delete my-allow-openvpn
```

Test port by openssl command

```console
openssl s_client -connect 1.2.3.4:2295
```

### client openvpn

[How To Set Up an OpenVPN Server on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-16-04)

#### server side prepare

> [How To Set Up an OpenVPN Server on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-16-04)

First do something on server side

Step 6

```console
cd ~/openvpn-ca
source vars
./build-key client2
....................................................................................+++
writing new private key to 'client2.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [US]:
State or Province Name (full name) [CA]:
Locality Name (eg, city) [SanFrancisco]:
Organization Name (eg, company) [Fort-Funston]:
Organizational Unit Name (eg, section) [MyOrganizationalUnit]:
Common Name (eg, your name or your server's hostname) [client2]:
Name [server]:client2
Email Address [me@myhost.mydomain]:
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
Using configuration from /home/<xxxxxxxx>/openvpn-ca/openssl-1.0.0.cnf
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
countryName           :PRINTABLE:'US'
stateOrProvinceName   :PRINTABLE:'CA'
localityName          :PRINTABLE:'SanFrancisco'
organizationName      :PRINTABLE:'Fort-Funston'
organizationalUnitName:PRINTABLE:'MyOrganizationalUnit'
commonName            :PRINTABLE:'client2'
name                  :PRINTABLE:'client2'
emailAddress          :IA5STRING:'me@myhost.mydomain'
Certificate is to be certified until Feb 22 14:11:05 2028 GMT (3650 days)
Sign the certificate? [y/n]:y
1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
aaray21cn@instance-1:~/openvpn-ca$
```

Step 10

```console
mkdir -p ~/client-configs/files
chmod 700 ~/client-configs/files
cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf ~/client-configs/base.conf
vi ~/client-configs/base.conf
```

```aconf
client
dev tun
proto tcp
remote [GLOUD EXTERNAL IP] 2295
resolv-retry infinite
nobind
user nobody
group nogroup
persist-key
persist-tun
ns-cert-type server
cipher AES-128-CBC
auth SHA256
key-direction 1
comp-lzo
verb 3
# script-security 2
# up /etc/openvpn/update-resolv-conf
# down /etc/openvpn/update-resolv-conf
```

```console
vi ~/client-configs/make_config.sh
```

```bash
#!/bin/bash

# First argument: Client identifier

KEY_DIR=~/openvpn-ca/keys
OUTPUT_DIR=~/client-configs/files
BASE_CONFIG=~/client-configs/base.conf

cat ${BASE_CONFIG} \
    <(echo -e '<ca>') \
    ${KEY_DIR}/ca.crt \
    <(echo -e '</ca>\n<cert>') \
    ${KEY_DIR}/${1}.crt \
    <(echo -e '</cert>\n<key>') \
    ${KEY_DIR}/${1}.key \
    <(echo -e '</key>\n<tls-auth>') \
    ${KEY_DIR}/ta.key \
    <(echo -e '</tls-auth>') \
    > ${OUTPUT_DIR}/${1}.ovpn
```

```console
chmod 700 ~/client-configs/make_config.sh
cd ~/client-configs
./make_config.sh client2
ls ~/client-configs/files
client2.ovpn
```

#### Transferring Configuration to Client Devices

[连接到 gcloud Linux 实例](https://cloud.google.com/compute/docs/instances/connecting-to-instance#standardssh)

```console
ssh-keygen -t rsa -f ~/.ssh/my-ssh-key -C aaray21cn
ssh -i ~/.ssh/my-ssh-key [USERNAME]@[EXTERNAL_IP_ADDRESS]
exit
scp -i ~/.ssh/my-ssh-key [USERNAME]@[EXTERNAL_IP_ADDRESS]:~/client-configs/files/client2.ovpn ~/
```

#### client (linux ubuntu)

```console
sudo apt-get update
sudo apt-get install openvpn
ls /etc/openvpn
vi ~/client2.ovpn
```

Uncomment the three lines we placed in to adjust the DNS settings if you were able to find an update-resolv-conf file:

```aconf
script-security 2
up /etc/openvpn/update-resolv-conf
down /etc/openvpn/update-resolv-conf
```

Add following 4 lines

```aconf
#by pass shadowsocks server address
route [GCLOUD EXTERNAL_IP_ADDRESS] 255.255.255.255 net_gateway
#use ss socks5
socks-proxy 127.0.0.1 1080
```

run following command

```console
sudo openvpn --config client2.ovpn

...
Sat Feb 24 23:05:00 2018 /sbin/ip route add 0.0.0.0/1 via 10.8.0.9
Sat Feb 24 23:05:00 2018 /sbin/ip route add 128.0.0.0/1 via 10.8.0.9
Sat Feb 24 23:05:00 2018 /sbin/ip route add 35.194.128.249/32 via 192.168.1.1
Sat Feb 24 23:05:00 2018 /sbin/ip route add 10.8.0.1/32 via 10.8.0.9
Sat Feb 24 23:05:00 2018 GID set to nogroup
Sat Feb 24 23:05:00 2018 UID set to nobody
Sat Feb 24 23:05:00 2018 Initialization Sequence Completed

```

### additional function, set PC as router

### Shadowsocks over KCPTUN

[KCPTUN](https://github.com/xtaci/kcptun)
[SS over KCP](https://gist.github.com/WispZhan/656b88bb866f39d5bee6450513c5f6bd)

* install kcptun on server side

download [build release](https://github.com/xtaci/kcptun/releases). Find the package for your system. [kcptun-linux-amd64-20171201.tar.gz](https://github.com/xtaci/kcptun/releases/download/v20171201/kcptun-linux-amd64-20171201.tar.gz), get the URL link `https://github.com/xtaci/kcptun/releases/download/v20171201/kcptun-linux-amd64-20171201.tar.gz`

```console
mkdir -p ~/tool/kcptun
cd ~/tool/kcptun
wget https://github.com/xtaci/kcptun/releases/download/v20171201/kcptun-linux-amd64-20171201.tar.gz
tar xzvf kcptun-linux-amd64-20171201.tar.gz
```

create `server-config.json` file as following content.
localaddr is the kcptun listen port. remoteaddr is the shadowsocks ip:port.

```json
{
"localaddr": ":4443",
"remoteaddr": "1.2.3.4:443",
"key": "passwordofkcptun",
"crypt": "salsa20",
"mode": "fast",
"conn": 1,
"autoexpire": 60,
"mtu": 1350,
"sndwnd": 128,
"rcvwnd": 1024,
"datashard": 5,
"parityshard": 5,
"dscp": 46,
"nocomp": true,
"acknodelay": false,
"nodelay": 0,
"interval": 40,
"resend": 0,
"nc": 0,
"sockbuf": 4194304,
"keepalive": 10
}
```

create `start.sh` as following.

```bash
#!/bin/bash
cd /home/realuser/tool/kcptun/
./server_linux_amd64 -c ./server-config.json > kcptun.log 2>&1 &
echo "Kcptun started."
```

create `stop.sh` as following.

```bash
#!/bin/bash
echo "Stopping Kcptun..."
PID=`ps -ef | grep server_linux_amd64 | grep -v grep | awk '{print $2}'`
if [ "" !=  "$PID" ]; then
echo "killing $PID"
kill -9 $PID
fi
echo "Kcptun stoped."
```

make `*.sh` executable.

```console
chmod +x *.sh
```

make kcptun autostart on server

add following in `/etc/rc.local` before `exit 0`

```bash
/home/realuser/tool/kcptun/start.sh &
```

* server firewall

open port on server if server has firewall enabled.

```console
sudo ufw allow 4443
```

* VPS firewall (google cloud)

KCP uses UDP protocol usually. Run following commands on gcloud console.

```console
gcloud compute firewall-rules list
gcloud compute firewall-rules create my-allow-kcp --allow tdp:4443,udp:4443 --description="kcptun"
```

* install kcptun on client side

Linux client (ubuntu 16.04 x64)

```console
mkdir -p ~/tool/kcptun
cd ~/tool/kcptun
wget https://github.com/xtaci/kcptun/releases/download/v20171201/kcptun-linux-amd64-20171201.tar.gz
tar xzvf kcptun-linux-amd64-20171201.tar.gz
```

create `client-config.json` file as following content.
localaddr is the kcptun listen port on local. remoteaddr is the remote kcptun server ip:port. Others should be same as server.

```json
{
"localaddr": ":4443",
"remoteaddr": "1.2.3.4:4443",
"key": "passwordofkcptun",
"crypt": "salsa20",
"mode": "fast",
"conn": 1,
"autoexpire": 60,
"mtu": 1350,
"sndwnd": 128,
"rcvwnd": 1024,
"datashard": 5,
"parityshard": 5,
"dscp": 46,
"nocomp": true,
"acknodelay": false,
"nodelay": 0,
"interval": 40,
"resend": 0,
"nc": 0,
"sockbuf": 4194304,
"keepalive": 10
}
```

create `start.sh` as following.

```bash
#!/bin/bash
cd /home/realuser/tool/kcptun/
./client_linux_amd64 -c ./client-config.json > kcptun.log 2>&1 &
echo "Kcptun started."
```

create `stop.sh` as following.

```bash
#!/bin/bash
echo "Stopping Kcptun..."
PID=`ps -ef | grep client_linux_amd64 | grep -v grep | awk '{print $2}'`
if [ "" !=  "$PID" ]; then
echo "killing $PID"
kill -9 $PID
fi
echo "Kcptun stoped."
```

make `*.sh` executable.

```console
chmod +x *.sh
```

make kcptun autostart on server

add following in `/etc/rc.local` before `exit 0`

```bash
/home/realuser/tool/kcptun/start.sh &
```

Other client please refer to [this post](https://gist.github.com/WispZhan/656b88bb866f39d5bee6450513c5f6bd)

* update shadowsocks client configuration file and restart shadowsocks

```json
"server": "127.0.0.1",
"server_port": 4443,
...
```

### how to make other PC use the router

#### simple way, the PC can set default gateway

TODO

#### the PC don't have permission to update default gateway

##### set wifi as AP

Need following CreateAP tool

> [CreateAP](https://github.com/oblique/create_ap)

start_ov_ss.sh

```bash
#!/bin/bash

#echo start kcptun
#/home/quxr/tool/kcptun/start.sh

echo start ssr
sudo /root/tool/ssr start

echo connect openvpn
sudo openvpn --config /home/quxr/client1.ovpn &

echo sleeping 15s
sleep 15

echo set iptables
sudo iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE
sudo iptables -A FORWARD -i tun0 -o enp3s0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i enp3s0 -o tun0 -j ACCEPT

echo start AP RAY_901
sudo create_ap wlp1s0 enp3s0 RAY_901 THE_WIFI_PASSWORD &
```

##### use 2nd router's

[2nd router](/2019/04/23/lede-lubuntu-eeepc901/#关于旁路由)

### Reference

> [OpenVPN over Shadowsocks](https://dcamero.azurewebsites.net/shadowsocks-openvpn.html)
> [openvpn over shadowsocks.txt](https://gist.github.com/wynemo/149ab6ef43f48b4ce9f32a5d5f868203)