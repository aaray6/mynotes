---
title: openvpn over ssr on gcloud
date: 2018-02-24 19:03:50
tags:
---
# How to setup OpenVPN over ssr on gcloud

## apply VPC (google cloud compute engine)

## shadowsocks server

## PC (install lubuntu on eeepc901)

## shadowsocks client

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

### gcloud F/W (gcloud)

### client openvpn

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
Using configuration from /home/aaray21cn/openvpn-ca/openssl-1.0.0.cnf
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

### how to make other PC use the router

#### simple way, the PC can set default gateway

#### the PC don't have permission to update default gateway

##### set wifi as AP

##### use 2nd router's DHCP

### Reference

> [OpenVPN over Shadowsocks](https://dcamero.azurewebsites.net/shadowsocks-openvpn.html)
> [openvpn over shadowsocks.txt](https://gist.github.com/wynemo/149ab6ef43f48b4ce9f32a5d5f868203)