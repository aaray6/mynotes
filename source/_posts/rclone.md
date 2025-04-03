---
title: rclone
date: 2022-09-15 20:23:42
tags:
---

用rclone在linux下mount webdav.

> rclone的项目在https://rclone.org/

## 安装

安装参考https://rclone.org/install/

我用的是

```console
sudo -v ; curl https://rclone.org/install.sh | sudo bash
```

重新执行上面脚本会更新最新版本。

安装之后用普通用户运行

```console
rclone config
```

进入交互式配置向导，生成配置文件~/.config/rclone/rclone.conf

文件内容如下

```conf
[aliyundrive]
type = webdav
url = http://127.0.0.1:8088
vendor = nextcloud
user = admin
pass = Ekxxxxxxxxxxxxxxxxxxxxx5Bh
```

配置中的aliyundrive是自己起的名字。后面mount的时候需要用到。
http://127.0.0.1:8088 是我用aliyundrive-webdav配置的服务器。把阿里云盘变成webdav服务。

vendor选择nextcloud是根据aliyundrive-webdav的说明

>https://github.com/messense/aliyundrive-webdav

```text
rclone
由于 rclone 请求时总是会以上一个请求 URL 作为 Referer, 使用 rclone 时请使用 Web 版 refresh token 或者启动 aliyundrive-webdav 时增加 --no-redirect 参数.

为了避免重复上传文件，使用 rclone 时推荐使用 Nextcloud WebDAV 模式，可以支持 sha1 checksums. 另外需要配合 --no-update-modtime 参数，否则 rclone 为了更新文件修改时间还是会强制重新上传。

举个例子：

rclone --no-update-modtime copy abc.pdf aliyundrive-nc://docs/
```

## mount/umount

mount

```console
mkdir -p ~/mnt/aliyundrive
rclone mount aliyundrive:/ ~/mnt/aliyundrive --cache-dir /tmp --vfs-cache-mode writes --allow-non-empty --no-update-modtime
```

umount

```console
fusermount -u ~/mnt/aliyundrive
```

注: *在~/tool/bin下建了两个脚本rclone_mount_aliyundrive.sh和rclone_umount_aliyundrive.sh包含以上两条命令*

## 设置开机自动运行

注: *需要找到方法确保服务在docker container "aliyunwebdav"之后运行。暂时disable以下user service,改用脚本手工启动停止rclone*

创建~/.config/systemd/user/rclone_aliyundrive.service文件

内容如下

```service
# User service for Rclone mounting
#
# Place in ~/.config/systemd/user/
# As your normal user, run 
#   systemctl --user daemon-reload
#   systemctl --user enable rclone_aliyundrive
#   systemctl --user start rclone_aliyundrive
#   systemctl --user stop rclone_aliyundrive


[Unit]
Description=rclone: Remote FUSE filesystem for cloud storage config aliyundrive
Documentation=man:rclone(1)
After=network-online.target
Wants=network-online.target 
AssertPathIsDirectory=%h/mnt/aliyundrive

[Service]
Type=notify
ExecStart=/usr/bin/rclone mount \
    --config=%h/.config/rclone/rclone.conf \
    --cache-dir /tmp \
    --vfs-cache-mode writes \
    --allow-non-empty \
    aliyundrive:/02_backup %h/mnt/aliyundrive
ExecStop=/bin/fusermount -u %h/mnt/aliyundrive

[Install]
WantedBy=default.target

```

以下命令
$ systemd-analyze --user verify rclone_aliyundrive.service
用来检查脚本
$  systemctl --user daemon-reload
reload
$ systemctl --user enable rclone_aliyundrive
enable,enable之后就可以开机自动启动
$ systemctl --user start rclone_aliyundrive
启动
$ systemctl --user stop rclone_aliyundrive
停止

## SMB

从1.60版本开始可以支持SMB / CIFS

```console
user@user-ThinkPad-X201:~/tool/bin$ sudo -v ; curl https://rclone.org/install.sh | sudo bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  4669  100  4669    0     0   3868      0  0:00:01  0:00:01 --:--:--  3868

Archive:  rclone-current-linux-amd64.zip
   creating: tmp_unzip_dir_for_rclone/rclone-v1.61.1-linux-amd64/
  inflating: tmp_unzip_dir_for_rclone/rclone-v1.61.1-linux-amd64/README.html  [text]  
  inflating: tmp_unzip_dir_for_rclone/rclone-v1.61.1-linux-amd64/rclone  [binary]
  inflating: tmp_unzip_dir_for_rclone/rclone-v1.61.1-linux-amd64/README.txt  [text]  
  inflating: tmp_unzip_dir_for_rclone/rclone-v1.61.1-linux-amd64/rclone.1  [text]  
  inflating: tmp_unzip_dir_for_rclone/rclone-v1.61.1-linux-amd64/git-log.txt  [text]  
正在删除 /usr/share/man 里的旧数据库条目...
正在处理 /usr/share/man 下的手册页...
正在删除 /usr/share/man/zh_CN 里的旧数据库条目...
正在处理 /usr/share/man/zh_CN 下的手册页...
正在删除 /usr/share/man/sk 里的旧数据库条目...
正在处理 /usr/share/man/sk 下的手册页...
正在删除 /usr/share/man/ro 里的旧数据库条目...
正在处理 /usr/share/man/ro 下的手册页...
正在删除 /usr/share/man/fr.UTF-8 里的旧数据库条目...
正在处理 /usr/share/man/fr.UTF-8 下的手册页...
正在删除 /usr/share/man/es 里的旧数据库条目...
正在处理 /usr/share/man/es 下的手册页...
正在删除 /usr/share/man/fi 里的旧数据库条目...
正在处理 /usr/share/man/fi 下的手册页...
正在删除 /usr/share/man/eo 里的旧数据库条目...
正在处理 /usr/share/man/eo 下的手册页...
正在删除 /usr/share/man/cs 里的旧数据库条目...
正在处理 /usr/share/man/cs 下的手册页...
正在删除 /usr/share/man/hu 里的旧数据库条目...
正在处理 /usr/share/man/hu 下的手册页...
正在删除 /usr/share/man/zh_TW 里的旧数据库条目...
正在处理 /usr/share/man/zh_TW 下的手册页...
正在删除 /usr/share/man/ru 里的旧数据库条目...
正在处理 /usr/share/man/ru 下的手册页...
正在删除 /usr/share/man/sr 里的旧数据库条目...
正在处理 /usr/share/man/sr 下的手册页...
正在删除 /usr/share/man/ko 里的旧数据库条目...
正在处理 /usr/share/man/ko 下的手册页...
正在删除 /usr/share/man/tr 里的旧数据库条目...
正在处理 /usr/share/man/tr 下的手册页...
正在删除 /usr/share/man/nl 里的旧数据库条目...
正在处理 /usr/share/man/nl 下的手册页...
正在删除 /usr/share/man/id 里的旧数据库条目...
正在处理 /usr/share/man/id 下的手册页...
正在删除 /usr/share/man/fr 里的旧数据库条目...
正在处理 /usr/share/man/fr 下的手册页...
正在删除 /usr/share/man/de 里的旧数据库条目...
正在处理 /usr/share/man/de 下的手册页...
正在删除 /usr/share/man/pt_BR 里的旧数据库条目...
正在处理 /usr/share/man/pt_BR 下的手册页...
正在删除 /usr/share/man/pl 里的旧数据库条目...
正在处理 /usr/share/man/pl 下的手册页...
正在删除 /usr/share/man/el 里的旧数据库条目...
正在处理 /usr/share/man/el 下的手册页...
正在删除 /usr/share/man/ja 里的旧数据库条目...
正在处理 /usr/share/man/ja 下的手册页...
正在删除 /usr/share/man/zh 里的旧数据库条目...
正在处理 /usr/share/man/zh 下的手册页...
正在删除 /usr/share/man/ca 里的旧数据库条目...
正在处理 /usr/share/man/ca 下的手册页...
正在删除 /usr/share/man/gl 里的旧数据库条目...
正在处理 /usr/share/man/gl 下的手册页...
正在删除 /usr/share/man/pt 里的旧数据库条目...
正在处理 /usr/share/man/pt 下的手册页...
正在删除 /usr/share/man/da 里的旧数据库条目...
正在处理 /usr/share/man/da 下的手册页...
正在删除 /usr/share/man/hr 里的旧数据库条目...
正在处理 /usr/share/man/hr 下的手册页...
正在删除 /usr/share/man/sl 里的旧数据库条目...
正在处理 /usr/share/man/sl 下的手册页...
正在删除 /usr/share/man/vi 里的旧数据库条目...
正在处理 /usr/share/man/vi 下的手册页...
正在删除 /usr/share/man/nb 里的旧数据库条目...
正在处理 /usr/share/man/nb 下的手册页...
正在删除 /usr/share/man/it 里的旧数据库条目...
正在处理 /usr/share/man/it 下的手册页...
正在删除 /usr/share/man/uk 里的旧数据库条目...
正在处理 /usr/share/man/uk 下的手册页...
正在删除 /usr/share/man/sv 里的旧数据库条目...
正在处理 /usr/share/man/sv 下的手册页...
正在删除 /usr/share/man/fr.ISO8859-1 里的旧数据库条目...
正在处理 /usr/share/man/fr.ISO8859-1 下的手册页...
正在删除 /usr/local/man 里的旧数据库条目...
正在处理 /usr/local/man 下的手册页...
正在删除 /usr/local/man/ja_JP.UTF-8 里的旧数据库条目...
正在处理 /usr/local/man/ja_JP.UTF-8 下的手册页...
正在删除 /usr/local/man/ja 里的旧数据库条目...
正在处理 /usr/local/man/ja 下的手册页...
0 个 man 子目录包含更新的手册页。
添加了 0 个手册页。
添加了 0 个孤立 cat 页面。
删除了 27 条旧数据库条目。

rclone v1.61.1 has successfully installed.
Now run "rclone config" for setup. Check https://rclone.org/docs/ for more details.

user@user-ThinkPad-X201:~/tool/bin$ 
user@user-ThinkPad-X201:~/tool/bin$ rclone config
Current remotes:

Name                 Type
====                 ====
aliyundrive          webdav

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> n

Enter name for new remote.
name> nas1

Option Storage.
Type of storage to configure.
Choose a number from below, or type in your own value.
 1 / 1Fichier
   \ (fichier)
 2 / Akamai NetStorage
   \ (netstorage)
 3 / Alias for an existing remote
   \ (alias)
 4 / Amazon Drive
   \ (amazon cloud drive)
 5 / Amazon S3 Compliant Storage Providers including AWS, Alibaba, Ceph, China Mobile, Cloudflare, ArvanCloud, DigitalOcean, Dreamhost, Huawei OBS, IBM COS, IDrive e2, IONOS Cloud, Liara, Lyve Cloud, Minio, Netease, RackCorp, Scaleway, SeaweedFS, StackPath, Storj, Tencent COS, Qiniu and Wasabi
   \ (s3)
 6 / Backblaze B2
   \ (b2)
 7 / Better checksums for other remotes
   \ (hasher)
 8 / Box
   \ (box)
 9 / Cache a remote
   \ (cache)
10 / Citrix Sharefile
   \ (sharefile)
11 / Combine several remotes into one
   \ (combine)
12 / Compress a remote
   \ (compress)
13 / Dropbox
   \ (dropbox)
14 / Encrypt/Decrypt a remote
   \ (crypt)
15 / Enterprise File Fabric
   \ (filefabric)
16 / FTP
   \ (ftp)
17 / Google Cloud Storage (this is not Google Drive)
   \ (google cloud storage)
18 / Google Drive
   \ (drive)
19 / Google Photos
   \ (google photos)
20 / HTTP
   \ (http)
21 / Hadoop distributed file system
   \ (hdfs)
22 / HiDrive
   \ (hidrive)
23 / In memory object storage system.
   \ (memory)
24 / Internet Archive
   \ (internetarchive)
25 / Jottacloud
   \ (jottacloud)
26 / Koofr, Digi Storage and other Koofr-compatible storage providers
   \ (koofr)
27 / Local Disk
   \ (local)
28 / Mail.ru Cloud
   \ (mailru)
29 / Mega
   \ (mega)
30 / Microsoft Azure Blob Storage
   \ (azureblob)
31 / Microsoft OneDrive
   \ (onedrive)
32 / OpenDrive
   \ (opendrive)
33 / OpenStack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ (swift)
34 / Oracle Cloud Infrastructure Object Storage
   \ (oracleobjectstorage)
35 / Pcloud
   \ (pcloud)
36 / Put.io
   \ (putio)
37 / QingCloud Object Storage
   \ (qingstor)
38 / SMB / CIFS
   \ (smb)
39 / SSH/SFTP
   \ (sftp)
40 / Sia Decentralized Cloud
   \ (sia)
41 / Storj Decentralized Cloud Storage
   \ (storj)
42 / Sugarsync
   \ (sugarsync)
43 / Transparently chunk/split large files
   \ (chunker)
44 / Union merges the contents of several upstream fs
   \ (union)
45 / Uptobox
   \ (uptobox)
46 / WebDAV
   \ (webdav)
47 / Yandex Disk
   \ (yandex)
48 / Zoho
   \ (zoho)
49 / premiumize.me
   \ (premiumizeme)
50 / seafile
   \ (seafile)
Storage> 38

Option host.
SMB server hostname to connect to.
E.g. "example.com".
Enter a value.
host> 192.168.1.1

Option user.
SMB username.
Enter a string value. Press Enter for the default (user).
user> user

Option port.
SMB port number.
Enter a signed integer. Press Enter for the default (445).
port> 

Option pass.
SMB password.
Choose an alternative below. Press Enter for the default (n).
y) Yes, type in my own password
g) Generate random password
n) No, leave this optional password blank (default)
y/g/n> y
Enter the password:
password:
Confirm the password:
password:

Option domain.
Domain name for NTLM authentication.
Enter a string value. Press Enter for the default (WORKGROUP).
domain> 

Edit advanced config?
y) Yes
n) No (default)
y/n> 

Configuration complete.
Options:
- type: smb
- host: 192.168.1.1
- pass: *** ENCRYPTED ***
Keep this "nas1" remote?
y) Yes this is OK (default)
e) Edit this remote
d) Delete this remote
y/e/d> y

Current remotes:

Name                 Type
====                 ====
aliyundrive          webdav
nas1                 smb

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q
user@user-ThinkPad-X201:~/tool/bin$ cd ~/mnt
user@user-ThinkPad-X201:~/mnt$ ls -l
总用量 8
drwxrwxr-x 1 user user    0 2月  13 12:32 aliyundrive
drwxrwxr-x 3 user user 4096 2月   9 08:48 CloudNAS
drwxrwxr-x 2 user user 4096 10月 16 08:27 nfs1
user@user-ThinkPad-X201:~/mnt$ mkdir nas1
user@user-ThinkPad-X201:~/mnt$ rclone mount nas1:/ ~/mnt/nas1
```
