---
layout: post
author: "Neo"
title: "Đơn giản hóa việc ghi chú khi chơi CTF"
date: "2025-06-15"
tags: [
    "blog"
]
categories: blog
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

Đúng như tiêu đề, bài viết này tập trung vào việc làm thế nào để việc ghi chú khi chơi CTF hay khi thực hành các bài labs được dễ dàng hơn, giúp cho việc tra cứu lại cũng như viết writeup được thuận tiện hơn. Mục đích chính của bài viết là để tôi có cái xem lại cũng như copy/paste cho nhanh mỗi khi cần setup lại môi trường, nếu bạn xem được bài viết này, hy vọng nó giúp ích được cho bạn trong quá trình chơi CTF và công việc.

Bài viết dựa trên hướng dẫn của ByteSized Security: [Prepnote: Simplify Your Hacking Workflow](https://www.youtube.com/watch?v=BgDBzp8267I), cảm ơn anh vì 1 công cụ rất hữu ích.

## Requirements

Các công cụ được sử dụng:

1. Obsidian: Thằng này để note kết quả của các phases, lưu thêm cả ảnh nếu cần. Đây sẽ là kết quả để tổng hợp viết writeup
2. Vscode: Tôi chọn vscode vì nó tiện, giao diện cũng quen thuộc, tôi không phải cài thêm công cụ khác. Có thể thay thế bằng bất kỳ công cụ offline này như Sublime Text hay CherryTree.
3. [Prepnote](https://github.com/0xDynamo/Prepnote): Công cụ do ByteSized phát triển, có chức năng tạo tự động các thư mục, các file theo cấu trúc của từng phases khi chơi HackTheBox machine hay OSCP Playground, etc.

## Obsidian

Đầu tiên là cài đặt Obsidian cho kali. Vào [trang chủ](https://obsidian.md/download) của Obsidian và tải bản Deb cho Kali.

```bash
┌──(neo㉿fs0ci3ty)-[~]
└─$ sudo dpkg -i tools/obsidian_1.8.10_amd64.deb
[sudo] password for neo:
Selecting previously unselected package obsidian.
(Reading database ... 328836 files and directories currently installed.)
Preparing to unpack .../obsidian_1.8.10_amd64.deb ...
Unpacking obsidian (1.8.10) ...
Setting up obsidian (1.8.10) ...
update-alternatives is /usr/bin/update-alternatives
update-alternatives: using /opt/Obsidian/obsidian to provide /usr/bin/obsidian (obsidian) in auto mode
Cache read/write disabled: interface file missing. (Kernel needs AppArmor 2.4 compatibility patch.)
Warning: unable to find a suitable fs in /proc/mounts, is it mounted?
Use --subdomainfs to override.
dpkg: error processing package obsidian (--install):
 installed obsidian package post-installation script subprocess returned error exit status 1
Processing triggers for hicolor-icon-theme (0.18-2) ...
Processing triggers for desktop-file-utils (0.28-1) ...
Errors were encountered while processing:
 obsidian
```

## Vscode

Trong bài hướng dẫn của ByteSized thì anh ta dùng Sublime Text, tuy nhiên tôi quen dùng Vscode hơn, và bạn cũng nên như vậy, quen cái gì thì cứ dùng cái đó.

Cài đặt Vscode trên kali cũng rất đơn giản

`sudo apt install code-oss -y`

## Prepnote

Clone repo này về máy

`git clone https://github.com/0xDynamo/Prepnote.git`

```bash
┌──(neo㉿fs0ci3ty)-[~/tools]
└─$ cd Prepnote

┌──(neo㉿fs0ci3ty)-[~/tools/Prepnote]
└─$ ll
total 32
-rwxr-xr-x 1 neo neo 9395 Jun 18 21:43 prepnote.sh
-rw-r--r-- 1 neo neo 3816 Jun 18 21:43 README.md
-rwxr-xr-x 1 neo neo 7030 Jun 18 21:43 setup_prepnote.sh
drwxr-xr-x 4 neo neo 4096 Jun 18 21:43 Template
-rw-r--r-- 1 neo neo 1469 Jun 18 21:43 template_config
```

Trong lần đầu tiên cài đặt tôi sẽ chạy `setup_prepnote.sh`

```bash
┌──(neo㉿fs0ci3ty)-[~/tools/Prepnote]
└─$ ./setup_prepnote.sh


      ____                                   _          ____                              __          _____      __
     / __ \__  ______  ____ _____ ___  ____ ( )_____   / __ \________  ____  ____  ____  / /____     / ___/___  / /___  ______
    / / / / / / / __ \/ __ / __ __ \/ __ \|// ___/  / /_/ / ___/ _ \/ __ \/ __ \/ __ \/ __/ _ \    \__ \/ _ \/ __/ / / / __ \
   / /_/ / /_/ / / / / /_/ / / / / / / /_/ / (__  )  / ____/ /  /  __/ /_/ / / / / /_/ / /_/  __/   ___/ /  __/ /_/ /_/ / /_/ /
  /_____/\__, /_/ /_/\__,_/_/ /_/ /_/\____/ /____/  /_/   /_/   \___/ .___/_/ /_/\____/\__/\___/   /____/\___/\__/\__,_/ .___/
      /____/                                                     /_/                                                /_/



Note: The setup will be quicker if you fill out the 'config.txt' file with your paths beforehand.
Do you wish to continue with the interactive setup? (Y/n) [Y]:
```

Có 1 lưu ý là sau khi trả lời hết các câu hỏi setup thì tất cả config của tôi sẽ được lưu vào file mới là config.txt, tôi chỉ cần lưu lại file này và dùng cho các lần cài đặt tiếp theo.

Bây giờ trước khi trả lời các câu hỏi setup, tôi cần hiểu rõ về cây thư mục sau khi tool tạo ra. Nó sẽ tương tự như bên dưới

```bash
.
├── labs
│   ├── gp
│   ├── htb
│   ├── oscp-ad
│   ├── oscp-challenge
│   ├── oscp-exam
│   └── oscp-training
└── platforms-vault
    ├── template
    │   ├── enum
    │   │   ├── external
    │   │   │   ├── 135 - msrpc.md
    │   │   │   ├── 139 - netbios-ssn.md
    │   │   │   ├── 22 - ssh.md
    │   │   │   ├── 389 - 636 - 3268 - 3269 - ldap.md
    │   │   │   ├── 443 - https.md
    │   │   │   ├── 445 - samba.md
    │   │   │   ├── 80 - http.md
    │   │   │   ├── Untitled 4.md
    │   │   │   ├── Untitled 5.md
    │   │   │   └── Untitled 6.md
    │   │   └── internal
    │   │       └── User 1.md
    │   ├── Exploit.md
    │   ├── General Information.md
    │   ├── Loot.md
    │   └── Privesc
    │       └── User 1.md
    └── Untitled.md
```

- `labs`: thư mục chứa các note theo từng nền tảng để dễ phân loại cũng như tìm kiếm. Đây cũng chính là workspace làm việc của tôi trên Vscode.
- `platforms-vault`: chứa template sử dụng cho Obsidian.

Và để có được cây thư mục này thì câu trả lời của tôi cho các câu hỏi sẽ như bên dưới

```bash
┌──(neo㉿fs0ci3ty)-[~/tools/Prepnote]
└─$ ./setup_prepnote.sh


      ____                                   _          ____                              __          _____      __
     / __ \__  ______  ____ _____ ___  ____ ( )_____   / __ \________  ____  ____  ____  / /____     / ___/___  / /___  ______
    / / / / / / / __ \/ __ / __ __ \/ __ \|// ___/  / /_/ / ___/ _ \/ __ \/ __ \/ __ \/ __/ _ \    \__ \/ _ \/ __/ / / / __ \
   / /_/ / /_/ / / / / /_/ / / / / / / /_/ / (__  )  / ____/ /  /  __/ /_/ / / / / /_/ / /_/  __/   ___/ /  __/ /_/ /_/ / /_/ /
  /_____/\__, /_/ /_/\__,_/_/ /_/ /_/\____/ /____/  /_/   /_/   \___/ .___/_/ /_/\____/\__/\___/   /____/\___/\__/\__,_/ .___/
      /____/                                                     /_/                                                /_/



Note: The setup will be quicker if you fill out the 'config.txt' file with your paths beforehand.
Do you wish to continue with the interactive setup? (Y/n) [Y]: y
Please provide the paths for your setup. Press Enter to keep the current/suggested value.

Enter the path to your Obsidian notes vault (current: not set): /home/neo/platforms/platforms-vault
The path '/home/neo/platforms/platforms-vault' does not exist. Would you like to create it? (y/n) [Y]: y
Enter the path where you want to store the template folder within your notes vault (current: not set): /home/neo/platforms/platforms-vault/template
The path '/home/neo/platforms/platforms-vault/template' does not exist. Would you like to create it? (y/n) [Y]: y
Enter the path for your general working directory (current: not set): /home/neo/platforms/labs
The path '/home/neo/platforms/labs' does not exist. Would you like to create it? (y/n) [Y]: y
Enter the path for your OSCP Exam working folder (current: not set): /home/neo/platforms/labs/oscp-exam
The path '/home/neo/platforms/labs/oscp-exam' does not exist. Would you like to create it? (y/n) [Y]: y
Enter the path for your OSCP Training working folder (current: not set): /home/neo/platforms/labs/oscp-training
The path '/home/neo/platforms/labs/oscp-training' does not exist. Would you like to create it? (y/n) [Y]:
Enter the path for your HackTheBox working folder (current: not set): /home/neo/platforms/labs/htb
The path '/home/neo/platforms/labs/htb' does not exist. Would you like to create it? (y/n) [Y]:
Enter the path for your Proving Grounds working folder (current: not set): /home/neo/platforms/labs/pg
The path '/home/neo/platforms/labs/pg' does not exist. Would you like to create it? (y/n) [Y]:
Enter the path for your OSCP Challenge Labs working folder (current: not set): /home/neo/platforms/labs/oscp-challenges
The path '/home/neo/platforms/labs/oscp-challenges' does not exist. Would you like to create it? (y/n) [Y]:
Enter the path for your OSCP AD Challenge Labs working folder (current: not set): /home/neo/platforms/labs/oscp-ad
The path '/home/neo/platforms/labs/oscp-ad' does not exist. Would you like to create it? (y/n) [Y]:
Default template directory found at /home/neo/tools/Prepnote/Template.
Copying the default template folder to: /home/neo/platforms/platforms-vault/template
Default template successfully copied to /home/neo/platforms/platforms-vault/template.

Debug: OBSIDIAN_NOTES_PATH='/home/neo/platforms/platforms-vault'
Debug: TEMPLATE_PATH='/home/neo/platforms/platforms-vault/template'
Debug: EXAM_PATH='/home/neo/platforms/labs/oscp-exam'
Debug: TRAINING_PATH='/home/neo/platforms/labs/oscp-training'
Debug: WORKING_DIR='/home/neo/platforms/labs'
Debug: HTB_PATH='/home/neo/platforms/labs/htb'
Debug: PG_PATH='/home/neo/platforms/labs/pg'
Debug: OSCP_PATH='/home/neo/platforms/labs/oscp-challenges'
Debug: OSCP_AD_PATH='/home/neo/platforms/labs/oscp-ad'


Setup completed. Configuration saved to ./config.txt.


You can run this script again to modify paths or manually edit ./config.txt if needed.
Would you like to create a symlink for 'prepnote.sh' and 'config.txt' in /usr/local/bin for easier access? (y/n) [N]: y
[sudo] password for neo:
Symlinks created successfully. You can now run 'prepnote' from anywhere.
```

```bash
┌──(neo㉿fs0ci3ty)-[~/tools/Prepnote]
└─$ prepnote --help


      ____                                   _          ____                              __          _____      __
     / __ \__  ______  ____ _____ ___  ____ ( )_____   / __ \________  ____  ____  ____  / /____     / ___/___  / /___  ______
    / / / / / / / __ \/ __ / __ __ \/ __ \|// ___/  / /_/ / ___/ _ \/ __ \/ __ \/ __ \/ __/ _ \    \__ \/ _ \/ __/ / / / __ \
   / /_/ / /_/ / / / / /_/ / / / / / / /_/ / (__  )  / ____/ /  /  __/ /_/ / / / / /_/ / /_/  __/   ___/ /  __/ /_/ /_/ / /_/ /
  /_____/\__, /_/ /_/\__,_/_/ /_/ /_/\____/ /____/  /_/   /_/   \___/ .___/_/ /_/\____/\__/\___/   /____/\___/\__/\__,_/ .___/
      /____/                                                     /_/                                                /_/


Usage: prepnote [OPTIONS] FOLDER_NAME
Options:
  -h    Copy template to 'HackTheBox' destination.
  -p    Copy template to 'Proving Grounds' destination.
  -o    Copy template to 'OSCP - Challenge Labs' destination.
  -a    Copy template to 'OSCP - AD Challenge Labs' destination (with three machine folders).
  -e    Set up an OSCP exam environment with AD and challenge lab machines.
  -t    Set up an OSCP training environment with AD and challenge lab machines.
  --help Display this help message.
```

Thử tạo 1 máy HTB

```bash
┌──(neo㉿fs0ci3ty)-[~/tools/Prepnote]
└─$ prepnote -h environment
Copying template to: /home/neo/platforms/platforms-vault/My Hacking Adventures/HackTheBox/Environment
Template copied successfully.
Preparing to create folders in: /home/neo/platforms/labs/htb/environment
Created: /home/neo/platforms/labs/htb/environment/enum
Created: /home/neo/platforms/labs/htb/environment/loot
Created: /home/neo/platforms/labs/htb/environment/privesc
Created: /home/neo/platforms/labs/htb/environment/exploit
Created: /home/neo/platforms/labs/htb/environment/enum/internal
Created: /home/neo/platforms/labs/htb/environment/enum/external
Folders and subfolders created successfully.
```

Xem lại cây thư mục

```bash
.
├── labs
│   ├── htb
│   │   └── environment
│   │       ├── enum
│   │       │   ├── external
│   │       │   └── internal
│   │       ├── exploit
│   │       ├── loot
│   │       └── privesc
│   ├── oscp-ad
│   ├── oscp-challenges
│   ├── oscp-exam
│   ├── oscp-training
│   └── pg
└── platforms-vault
    ├── My Hacking Adventures
    │   └── HackTheBox
    │       └── Environment
    │           ├── enum
    │           │   ├── external
    │           │   │   ├── 135 - msrpc.md
    │           │   │   ├── 139 - netbios-ssn.md
    │           │   │   ├── 22 - ssh.md
    │           │   │   ├── 389 - 636 - 3268 - 3269 - ldap.md
    │           │   │   ├── 443 - https.md
    │           │   │   ├── 445 - samba.md
    │           │   │   ├── 80 - http.md
    │           │   │   ├── Untitled 4.md
    │           │   │   ├── Untitled 5.md
    │           │   │   └── Untitled 6.md
    │           │   └── internal
    │           │       └── User 1.md
    │           ├── Exploit.md
    │           ├── General Information.md
    │           ├── Loot.md
    │           └── Privesc
    │               └── User 1.md
    └── template
        ├── enum
        │   ├── external
        │   │   ├── 135 - msrpc.md
        │   │   ├── 139 - netbios-ssn.md
        │   │   ├── 22 - ssh.md
        │   │   ├── 389 - 636 - 3268 - 3269 - ldap.md
        │   │   ├── 443 - https.md
        │   │   ├── 445 - samba.md
        │   │   ├── 80 - http.md
        │   │   ├── Untitled 4.md
        │   │   ├── Untitled 5.md
        │   │   └── Untitled 6.md
        │   └── internal
        │       └── User 1.md
        ├── Exploit.md
        ├── General Information.md
        ├── Loot.md
        └── Privesc
            └── User 1.md

28 directories, 30 files
```

Vậy là xong, bây giờ thì cứ đến phần nào tôi sẽ tạo note cho riêng phần đó, dễ tra cứu lại cũng như tổng hợp lại thành writeup.

Một lần nữa cảm ơn ByteSized Security vì 1 công cụ hữu ích.







