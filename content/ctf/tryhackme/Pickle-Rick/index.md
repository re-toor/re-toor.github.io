---
title: "Tryhackme - Pickle Rick"
date: "2021-09-11"
tags: [
    "web",
    "linux",
    "html",
]
---

*"This Rick and Morty themed challenge requires you to exploit a webserver to find 3 ingredients that will help Rick make his potion to transform himself back into a human from a pickle."*

Hôm nay tôi sẽ giải CTF [Tryhackme - Pickle Rick](https://tryhackme.com/room/picklerick). Géc gô!!!

Vẫn như thông thường thôi, việc đầu tiên cần làm là quét cổng. Tôi sẽ dùng rustscan cho nhanh. 

![scan port with rustscan](scan-port.png)
Chúng ta có 2 port đang mở:
-   port 22 cổng ssh trên ubuntu server
-   port 80 với Apache httpd 2.4.18 trên ubuntu server

Cùng check qua site với port 80 nào.

![global site](site.png)

Hmmm... Cũng không có gì đặc biệt lắm. Thử check qua source xem sao.

Tôi có thêm 1 link ảnh và 1 username.

![username](username.png?style=centerme)

Thường thì để tránh mất thời gian tôi sẽ check thử qua các uri phổ biến: robots.txt, admin, login.php, upload.php, vân vân mây mây...

Sau 1 lúc loay hoay thì tôi có thêm 2 uri là _robots.txt_ và _login.php_.

Và với _robots.txt_ "huyền thoại", tôi có thêm 1 dòng text "_Wubbalubbadubdub_" mà tôi cũng chưa hiểu lắm nhưng thôi cứ lưu lại, coi như hint.

![login-site](login-site.png)
Với login site, tôi đã thử và SQLi và XSS nhưng không có gì đặc sắc lắm.

Vậy là chúng ta có 1 site login, có 1 username và 1 đoạn text. Sao không thử lấy đoạn text kia làm password?
Bingo!!!

![dashboard](dashboard.png)

Ta có 1 dashboard đơn giản và chỉ dùng được mục commands, các mục khác cần phải đăng nhập bằng "REAL rick" chăng???

Quay lại Command Panel, tôi nhận ra nó giống như 1 terminal kết nối server khi nhập "whoami" và nó trả lại "www-data".

Thử list file và tôi có 1 file txt khá đặc biệt.

![list file](dir%20file.png)
Và vì nó nằm trong thư mục web nên thử mở nó như 1 uri, chúng ta có flag đầu tiên.

"ls" 1 lúc thì tôi tìm được 2 user ở trong _/home_ là __rick__ và __ubuntu__, trong user __rick__ chúng ta tìm được file chứa flag thứ 2.

Tuy nhiên chúng ta không thể dùng "cat" để mở được file này. Sau một lúc google thì tôi tìm được "less" cũng tương tự như "cat".

Sử dụng less và chúng ta có flag thứ 2.

![second flag](second%20flag.png)
Đến đây thì thực sự tôi chưa nghĩ ra được cách tìm flag thứ 3. Tôi đã thử tìm file bằng tên với hi vọng tên file flag thứ 3 cũng giống như flag 2: second ingredients nhưng không có gì.

Thông thường, để có được flag cuối chúng ta phải leo thang đặc quyền lên root, và đa số flag cuối nằm trong thư mục root.

Trong 1 phút ngáo ngơ tôi quên mất sudo -l =))))

![sudo -l](sudo%20-l.png)
Điều này có nghĩa là chúng ta có toàn quyền root mà không cần password.

![third flag](ls%20root.png)
Vậy là đã xong. Rooted.

