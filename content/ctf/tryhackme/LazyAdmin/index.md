---
title: "Tryhackme - LazyAdmin"
date: "2021-09-22"
tags: [
    "web",
    "linux",
    "ssh",
    "exploit-db"
]
---

Hôm nay tôi sẽ giải CTF [Tryhackme - LazyAdmin](https://tryhackme.com/room/lazyadmin). Géc gô!!!

Vẫn như thông thường thôi, việc đầu tiên cần làm là quét cổng. Tôi sẽ dùng [Rustscan](https://github.com/RustScan/RustScan) cho nhanh và thêm vài tùy chọn của nmap.

![scan-port](scan-port.png?style=centerme)

Chúng ta có 2 port đang mở:
-   port 22 chạy ssh trên ubuntu server
-   port 80 với Apache httpd 2.4.18 trên ubuntu server

Với header này có vẻ như web chỉ là default page Apache. Kiểm tra source web cũng không có gì đặc biệt.

Thử [dirsearch](https://github.com/maurosoria/dirsearch) để tìm web path nào. Thằng này cú pháp đơn giản hơn [gobuster](https://github.com/OJ/gobuster) vì nó có sẵn wordlist khá đầy đủ và còn được cập nhật.

![dirsearch](dirsearch.png?style=centerme)

Thử vào path và tôi có 1 site index của basic-cms sweetrice. 

![sweetrice](sweetrice.png)

Thử tìm trên [exploit-db](https://www.exploit-db.com/) và tôi tìm được exploit upload file: [40716](https://www.exploit-db.com/exploits/40716). Xem qua exploit này thì tôi thấy có vài path ở đây. 

![path](path.png?style=centerme)

Vậy có thể là sau __/content/__ vẫn còn có các path nữa.

Thử thêm 1 lần dirsearch nữa xem sao.

Và tôi có thêm các path của site sweetrice, trong đó có path __/as/__ dẫn đến site login và path __/inc/__ chứa các file hệ thống. Tôi để ý có mysql_backup ở đây. Thử tải và kiểm tra xem có user và password không nào.

![user-pass](user-pass.png?style=centerme)

Đây rồi! Tôi có username và password với hash md5. Vào [Crackstations](https://crackstation.net/) để unhash password. Login thôi.

![site](site.png)

Dashboard này nhiều thứ quá, tôi nhớ ra exploit lúc nãy cần username và password để upload file. Vậy thì thử thôi upload reverse shell nào.

![upload](upload.png?style=centerme)

Được rồi này. Điều lưu ý ở đây là cần phải đổi .php sang .php5, nếu để .php thì server không cho upload.

Bật netcat lên với listen port đã tạo trong shell: `nc -lnvp 1234`

![user-flag](user-flag.png)

Config lại session với python.

`python3 -c ‘import pty;pty.spawn(”/bin/bash”)’`

`export TERM=xterm`

`Ctrl+Z`
      
`stty raw -echo; fg`

Tìm trong thư mục /home/ có user *itguy*, và tôi tìm được user flag ở đây.

Bây giờ thì thử tìm xem có thư mục nào chạy bằng quyền root không.

![sudo-l](sudo-l.png?style=centerme)

Qua [GTFOBins](https://gtfobins.github.io/) tìm __perl__ nào.

![gtfobins](gtfobins.png?style=centerme)

Sử dụng **sudo** nhưng không được vì yêu cầu pass của user web. 

![cannot](cannot.png?style=centerme)

Tôi còn 1 file với quyền root nữa mà không cần pass là backup.pl. 

![backup](backup.png?style=centerme)

Xem nội dung file __copy.sh__ thì nó là 1 reverse shell khác. Sửa file này để nó trỏ về máy local của tôi.

![get-shell](get-shell.png?style=centerme)

Dùng netcat để get shell

![rooted](rooted.png?style=centerme)

Rooted.

Tìm được rồi!
ftpuser:ftpuser@123






