---
layout: post
author: Neo
title: Hackthebox - Signed
image: /assets/img/2025-11-02-HTB-DarkZero/intro.png
date: 2025-11-02
tags:
  - windows
  - hackthebox
categories: ctf
---
0. this unordered seed list will be replaced by toc as unordered list
{:toc}

> As is common in real life pentests, you will start the DarkZero box with credentials for the following account john.w / RFulUtONCOL!

## Reconnaissance and Scanning

```bash
rustscan -a 10.10.11.90 -- -sC -sV -oN nmap
```

```bash
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-11-03 15:45:50Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: darkzero.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.darkzero.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.darkzero.htb
| Not valid before: 2025-07-29T11:40:00
|_Not valid after:  2026-07-29T11:40:00
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: darkzero.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC01.darkzero.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.darkzero.htb
| Not valid before: 2025-07-29T11:40:00
|_Not valid after:  2026-07-29T11:40:00
1433/tcp open  ms-sql-s      Microsoft SQL Server 2022 16.00.1000.00; RTM
| ms-sql-ntlm-info:
|   10.10.11.89:1433:
|     Target_Name: darkzero
|     NetBIOS_Domain_Name: darkzero
|     NetBIOS_Computer_Name: DC01
|     DNS_Domain_Name: darkzero.htb
|     DNS_Computer_Name: DC01.darkzero.htb
|     DNS_Tree_Name: darkzero.htb
|_    Product_Version: 10.0.26100
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2025-11-01T02:35:23
|_Not valid after:  2055-11-01T02:35:23
|_ssl-date: 2025-11-03T15:47:15+00:00; +6h32m10s from scanner time.
| ms-sql-info:
|   10.10.11.89:1433:
|     Version:
|       name: Microsoft SQL Server 2022 RTM
|       number: 16.00.1000.00
|       Product: Microsoft SQL Server 2022
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
2179/tcp open  vmrdp?
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: darkzero.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.darkzero.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.darkzero.htb
| Not valid before: 2025-07-29T11:40:00
|_Not valid after:  2026-07-29T11:40:00
|_ssl-date: TLS randomness does not represent time
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: darkzero.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.darkzero.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.darkzero.htb
| Not valid before: 2025-07-29T11:40:00
|_Not valid after:  2026-07-29T11:40:00
|_ssl-date: TLS randomness does not represent time
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time:
|   date: 2025-11-03T15:46:38
|_  start_date: N/A
|_clock-skew: mean: 6h32m09s, deviation: 0s, median: 6h32m09s
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled and required
```

Phân tích một chút về kết quả scan

- **DNS (53)** - Simple DNS Plus
- **Kerberos (88)** - Windows Kerberos authentication
- **LDAP (389/636/3268/3269)** - Active Directory services
- **SMB (139/445)** - File sharing
- **MSSQL (1433)** - Microsoft SQL Server 16.00.1000.00
- **WinRM (5985)** - Remote management
- **RPC (135, various high ports)** - Remote procedure calls

Thêm domain và file host

```bash
sudo nano /etc/hosts
```

```bash
10.10.11.89     darkzero.htb    DC01.darkzero.htb
```
## Enumeration and Gaining access

Sử dụng `netexec` để check các service với user được cung cấp

```bash
┌──(neo㉿fs0ci3ty)-[~/htb/machines/DarkZero]
└─$ netexec smb 10.10.11.89 -u john.w -p 'RFulUtONCOL!' --shares
SMB         10.10.11.89     445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:darkzero.htb) (signing:True) (SMBv1:False) 
SMB         10.10.11.89     445    DC01             [+] darkzero.htb\john.w:RFulUtONCOL! 
SMB         10.10.11.89     445    DC01             [*] Enumerated shares
SMB         10.10.11.89     445    DC01             Share           Permissions     Remark
SMB         10.10.11.89     445    DC01             -----           -----------     ------
SMB         10.10.11.89     445    DC01             ADMIN$                          Remote Admin
SMB         10.10.11.89     445    DC01             C$                              Default share
SMB         10.10.11.89     445    DC01             IPC$            READ            Remote IPC
SMB         10.10.11.89     445    DC01             NETLOGON        READ            Logon server share 
SMB         10.10.11.89     445    DC01             SYSVOL          READ            Logon server share 
```

Có vẻ như không có gì để khai thác. Sử dụng `dig` để khai thác DNS

```bash
┌──(neo㉿fs0ci3ty)-[~/htb/machines/DarkZero]
└─$ dig @DC01.darkzero.htb ANY darkzero.htb

; <<>> DiG 9.20.11-4+b1-Debian <<>> @DC01.darkzero.htb ANY darkzero.htb
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58136
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
;; QUESTION SECTION:
;darkzero.htb.                  IN      ANY

;; ANSWER SECTION:
darkzero.htb.           600     IN      A       172.16.20.1
darkzero.htb.           600     IN      A       10.10.11.89
darkzero.htb.           3600    IN      NS      dc01.darkzero.htb.
darkzero.htb.           3600    IN      SOA     dc01.darkzero.htb. hostmaster.darkzero.htb. 437 900 600 86400 3600

;; ADDITIONAL SECTION:
dc01.darkzero.htb.      3600    IN      A       10.10.11.89
dc01.darkzero.htb.      3600    IN      A       172.16.20.1

;; Query time: 107 msec
;; SERVER: 10.10.11.89#53(DC01.darkzero.htb) (TCP)
;; WHEN: Mon Nov 03 16:42:02 +07 2025
;; MSG SIZE  rcvd: 171
```

Từ đây nhận thấy domain có internal network `172.16.20.1/16`, lưu lại đây để dùng sau trong trường hợp cần làm pivot point.

