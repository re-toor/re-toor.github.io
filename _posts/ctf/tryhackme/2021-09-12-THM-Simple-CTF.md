---
layout: post
author: "Neo"
title: "Tryhackme - Simple CTF"
date: "2021-09-12"
tags: [
    "tryhackme",
    "ftp",
    "web",
    "linux",
    "php",
    "ssh"
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/THM-Simple-CTF/intro.png)

Xin chào, Lẩu đây. Hôm nay tôi sẽ giải CTF [Tryhackme - Simple CTF](https://tryhackme.com/room/easyctf). Géc gô!!!

Vẫn như thường lệ, việc đầu tiên cần làm là quét cổng. Tôi sẽ dùng rustscan cho nhanh. Ưu điểm của thằng này là quét full port rất nhanh, ngoài ra nó cũng có thể tích hợp nmap.

![scan-port](/assets/img/THM-Simple-CTF/scan-port.png?style=centerme)

Có 3 port đang mở:
- Port 21 chạy ftp-vsftpd 3.0.3
- Port 80 chạy web service với Apache 2.4.18 trên Ubuntu
- Port 2222 chạy ssh service

Với port 21, chúng ta có thể login với Anonymous. 

Tuy nhiên khi tôi thử list file, server timeout. Tôi thử tìm trên exploit-db để xem có exploit nào cho phiên bản 3.0.3 này không, chỉ có 1 exploit về DoS.

Để lại đó vậy. Chúng ta sẽ check qua web xem sao.

![port-80](/assets/img/THM-Simple-CTF/port80.png?style=centerme)

Tôi có 1 robots.txt với 2 disallow enties. Nhưng có vẻ như không vào được. 

Tuy nhiên tôi tìm được 1 cái tên ở đây: mike. 

![mike](/assets/img/THM-Simple-CTF/mike.png?style=centerme)

Tôi cũng không chắc nhưng cứ lưu lại vậy.

Tôi thấy có chút kỳ lạ ở entry thứ 2 nên đã check gg. Ầu, ra rồi! Openemr là một phần mềm quản lý y tế và phiên bản 5.0.1.3 có __CVE-2018-15139__. Nhưng có vẻ như không có tác dụng lắm do không có username và password.

Lại dirsearch để tìm directory thôi.

Tôi tìm ra 1 path mới dẫn đến 1 cms khác - CMS Made Simple với phiên bản 2.2.8. 

![other-dir](/assets/img/THM-Simple-CTF/other-dir.png?style=centerme)

Thử check qua CVE và tôi có CVE-2019-9053 sử dụng __SQLi__ để login mà không cần password. Check searchsploit và tôi có sploit 46635.

![46635](/assets/img/THM-Simple-CTF/46635.png?style=centerme)

Check qua exploit thì tôi tự hỏi tại sao vào năm 2019 mà nghệ sĩ này vẫn còn dùng python2 nhỉ??? Anyway, thử exploit nào.

![exploit](/assets/img/THM-Simple-CTF/exploit.png?style=centerme)

Và tôi tìm được password: __secret__. Login vào và dạo qua 1 vòng để tìm xem có chỗ nào upload shell được không, nhưng site này nhiều thứ quá, tôi hơi lười. 

![site-cms](/assets/img/THM-Simple-CTF/site-cms.png?style=centerme)

Tôi nhớ ra mình còn port 2222 ssh nữa mà.

Thử ssh với user và password ở port 2222 xem sao.

![ssh](/assets/img/THM-Simple-CTF/ssh.png?style=centerme)

Tada! Ở đây tôi có __user.txt__

Ở thư mục /home, tôi có 2 user __mitch__ và __sunbath__. 

Bây giờ thử tìm xem có cái gì chạy với root không.

![sudo-l](/assets/img/THM-Simple-CTF/sudo-l.png?style=centerme)

Vim chạy với quyền root không cần password. Qua [GTFOBins](https://gtfobins.github.io/) tìm __vim__ nào.

Chúng ta có 2 cách để lấy được root flag. 1 là dùng vim để mở file luôn, cách này thì phải xác định được vị trí của file, và thường thì root flag sẽ nằm trong thư mục root rồi. Còn cách thứ 2 là chiếm quyền root thôi.

Với cách 1 tôi chỉnh sửa file như thường thôi

![sudo-vim](/assets/img/THM-Simple-CTF/sudo-vim.png?style=centerme)

Với cách 2 sử dụng GTFOBins Sudo 

![rooted](/assets/img/THM-Simple-CTF/rooted.png?style=centerme)

Đã xong. Rooted!!!
