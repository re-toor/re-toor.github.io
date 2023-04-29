---
layout: post
author: "Neo"
title: "Tryhackme - Bounty Hacker"
date: "2021-09-15"
tags: [
    "tryhackme",
    "ftp",
    "web",
    "linux",
    "ssh",
    "hydra",
    "brute-force"
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![Intro](/assets/img/THM-Bounty-Hacker/Intro.png)

Xin chào, Lẩu đây. Hôm nay tôi sẽ giải CTF [Tryhackme - Bounty Hacker](https://tryhackme.com/room/cowboyhacker). Géc gô!!!

Vẫn như thông thường thôi, việc đầu tiên cần làm là quét cổng. Tôi sẽ dùng rustscan cho nhanh và thêm vài tùy chọn của nmap.

![scan-port](/assets/img/THM-Bounty-Hacker/scan-port.png?style=center)

Có 3 port đang mở:
- Port 21 chạy ftp-vsftpd 3.0.3 có thể login với Anonymous
- Port 22 chạy ssh service trên Ubuntu server
- Port 80 chạy web service với Apache 2.4.18 trên Ubuntu

Tôi thử login vào ftp server với anonymous.

![ftp](/assets/img/THM-Bounty-Hacker/ftp-server.png?style=centerme)

Tôi có 2 file txt ở đây. Clone nó về máy để xem có thông tin gì không.

`get locks.txt /home/neo/locks.txt`

`get task.txt /home/neo/task.txt`

![task-txt](/assets/img/THM-Bounty-Hacker/task-txt.png?style=centerme)

Tôi có 1 cái tên ở đây: __lin__

Với file __locks.txt__, tôi có 1 list mà tôi nghĩ sẽ dùng để brute-force. Vậy là tôi có 1 cái tên và 1 wordlist. Thử vào web xem có nơi nào để login không.

Tôi có 1 trang html về 1 cuộc hội thoại và vài cái tên. Không có gì đặc biệt hay kỳ lạ ở đây, mấy dir cơ bản cũng không có. Tôi dùng dirsearch để tìm dir nhưng cũng không thu được gì.

Vậy thì login vào đâu? Chỉ còn ssh thôi. Hail Hydra!!!

![Hydra](/assets/img/THM-Bounty-Hacker/hydra.png?style=centerme)

Login vào ssh và tôi có user flag nằm trong desktop.

![user-flag](/assets/img/THM-Bounty-Hacker/user-flag.png?style=centerme)

Thử tìm xem có thư mục nào chạy với quyền root không.

![sudo-l](/assets/img/THM-Bounty-Hacker/sudo-l.png?style=centerme)

Vào [GTFOBins](https://gtfobins.github.io/) xem có tar trong đó không.

![gtfobins](/assets/img/THM-Bounty-Hacker/gtfobins.png?style=centerme)

Thử xem có get root được không nào.

![rooted](/assets/img/THM-Bounty-Hacker/rooted.png?style=centerme)

Done.