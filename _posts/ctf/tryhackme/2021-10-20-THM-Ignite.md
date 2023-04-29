---
layout: post
author: "Neo"
title: "Tryhackme - Ignite"
date: "2021-10-20"
tags: [
    "tryhackme",
    "web",
    "linux",
    "exploit-db",
    "fuel-cms"
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/THM-Ignite/intro.png)

Xin chào, Lẩu đây. Hôm nay tôi sẽ giải CTF [Tryhackme - Ignite](https://tryhackme.com/room/ignite). Géc gô!!!

Vẫn như thông thường thôi, việc đầu tiên cần làm là quét cổng. 

![scan-port](/assets/img/THM-Ignite/scan-port.png?styel=centerme)

Tôi chỉ có 1 port đang mở là port 80 chạy Apache httpd 2.4.18 trên Ubuntu server. 

Với http-header *Fuel CMS*, có nghĩa là site này dựng trên cms fuel phiên bản 1.4. Trong file robots.txt có 1 entry là __/fuel__. Path này sẽ điều hướng web đến trang login của fuel cms.

![login-site](/assets/img/THM-Ignite/login-site.png?styel=centerme)

Tôi thử login vào site bằng username và password mặc định: admin-admin.

![dashboard](/assets/img/THM-Ignite/dashboard.png?styel=centerme)

Site dashboard cũng không có gì đặc biệt lắm. Thực ra thì khi nhìn thấy fuel cms tôi đã nghĩ đến việc tìm [exploit-db](https://exploit-db.com) hoặc searchsploit, nhưng vì được cho login từ đầu nên bây giờ tôi mới thử.

![searchsploit](/assets/img/THM-Ignite/searchsploit.png?styel=centerme)

Với 3 exploit RCE này, tôi chọn cái thứ 3 vì nó là python code và chạy trên php.

![50477](/assets/img/THM-Ignite/50477.png?styel=centerme)

Có shell rồi này. Tôi sẽ dùng python reverse shell để trỏ về máy local. Nhưng có vẻ là không được, kể cả các shell khác.

À! Còn cách khác. Tôi sẽ upload file shell php từ máy local.

Tạo http server từ máy local.

![http-server](/assets/img/THM-Ignite/http-server.png?styel=centerme)

Sau đó dùng wget để tải file từ máy local về remote server.

![get-shell](/assets/img/THM-Ignite/get-shell.png?styel=centerme)

Bật netcat và mở file trên browser. `http://MACHIEN-IP/shell.php`

![shell](/assets/img/THM-Ignite/shell.png?styel=centerme)

Truy cập user www-data và tôi có user flag.

Bây giờ đến bước leo thang đặc quyền. Tôi đã thử vài thứ đơn giản như sudo -l hay crontab đều không có kết quả gì. Vì vậy, tôi sẽ tạo http server bằng python và wget [linPEAS](https://github.com/carlospolop/PEASS-ng).

![linpeas](/assets/img/THM-Ignite/linpeas.png?styel=centerme)

Dạo quanh 1 đống vô cùng nhiều thứ này thì tôi tìm được vài thông tin trong databse fuel_schema. 

Nhập thử pass này với root, xong.

![rooted](/assets/img/THM-Ignite/rooted.png?styel=centerme)

Rooted!


