---
title: "Viết Decoders và Rules cho hệ thống Wazuh - Part 1"
date: "2023-06-01"
tags: 
   - siem
---

## Để làm gì?

Wazuh là hệ thống log tập trung, nó sẽ thu thập log từ tất cả các nguồn được xác định, phân tích log và từ đó đưa ra thông tin hoặc cảnh báo cho người quản trị. Tuy nhiên, để Wazuh có thể hiểu và phân tích được những gì nó đã nhận thì cần chuyển đổi những dữ liệu đó sang dạng ngôn ngữ của Wazuh.

Log là một dạng nhật ký có cấu trúc, ghi lại liên tục các thông báo về hoạt động của cả hệ thống hoặc của các dịch vụ được triển khai trên hệ thống và file tương ứng. Tùy vào các hệ thống và các tiến trình khác nhau mà log sinh ra sẽ được định nghĩa theo các trường, các thông tin trong log sẽ tương ứng với giá trị của các trường đó.

Ví dụ 

```python
Dec 25 20:45:02 MyHost example[12345]: User 'admin' logged from '192.168.1.100'
```

Khi nhìn vào log này tôi có thể định nghĩa ra các các trường cố định ứng với giá trị thay đổi như `time`:`Dec 25 20:45:02`, `user`:`admin`, `ip`:`192.168.1.100`.

Wazuh cũng vậy, nó dựa vào các trường được định nghĩa để có thể hiểu và phân tích được log mà nó đã nhận. Và việc của chúng ta là dựa vào những log mà hệ thống sinh ra, định nghĩa chúng thành các trường để Wazuh hiểu được và đưa ra cảnh báo dựa trên nó. Hiểu đơn giản là dịch lại các ngôn ngữ khác thành ngôn ngữ của Wazuh để nó hiểu được và giao tiếp lại được với chúng ta.

Để làm được điều này, ta cần quan tâm 2 phần: ***Decoders*** và ***Rules***

## Decoders

### Là gì? Để làm gì?

Hiểu đơn giản thì ***decoders*** là trình biên dịch log để Wazuh đọc được và hiểu được log đó.

***Decoders*** được viết dưới dạng XML.

Trước tiên, để viết được thì ta cần hiểu các tùy chọn được sử dụng để cấu hình nên các ***decoder***. [Documentation của Wazuh](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/decoders.html) đã định nghĩa đầy đủ các tùy chọn có thể sử dụng. Nhưng trong bài này tôi sẽ chỉ liệt kê những tùy chọn quan trọng hay dùng nhất.

| Option | Values | Description |
| ------ | ------ | ----------- |
| [decoder](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/decoders.html#decoder) | Bất kỳ | Tên decoder |
| [parent](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/decoders.html#parent) | Tên decoder | Xác định decoder nào là cha để các thành phần của decoder con sẽ tham chiếu đến nó |
| [program_name](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/decoders.html#program-name) | [regex](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html#os-regex-syntax),[sregex](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html#sregex-os-match-syntax) hoặc [pcre2](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html#pcre2-syntax) | Đặt tên chương trình làm điều kiện để áp dụng bộ giải mã |
| [prematch](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/decoders.html#prematch) | [regex](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html#os-regex-syntax) hoặc [pcre2](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html#pcre2-syntax) | Đặt biểu thức chính quy làm điều kiện để áp dụng bộ giải mã |
| [regex](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/decoders.html#regex-decoders) | [regex](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html#os-regex-syntax) hoặc [pcre2](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html#pcre2-syntax) | Tìm các trường và định nghĩa chúng |
| [order](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/decoders.html#order) | [order table](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/decoders.html#decoders-order) | Xác định tên của các trường vừa được định nghĩa |
| [type](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/decoders.html#type) | [type table](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/decoders.html#decoders-type) | Phân loại các log theo nhóm |

Các tùy chọn khác có thể được dùng trong các trường hợp nhất định tùy thuộc vào đặc điểm nội dung của các loại log có trong hệ thống.

### Viết như thế nào?

Có thể thấy việc đầu tiên khi viết ***decoders*** là phải tạo `decoder` cha bên trong có chứa `program_name`, các `decoders` phía sau sẽ gắn `parent` là tên của `decoder` cha và thêm các options `prematch`, `regex`, `order`,...

Tôi sẽ lấy ví dụ về mẫu log có sẵn trong tập ***decoders*** mặc định của Wazuh

```python
May 10 11:13:40 jitauat sshd[112623]: Accepted password for root from 10.10.3.100 port 8266 ssh2
```

Log này có nội dung về đăng nhập thành công dịch vụ ssh với user *root* từ IP *10.10.3.100* port *8266*. Phân tích log để xác định các trường

- `May 10 11:13:40`: thời điểm login vào ssh
- `jitauat`: tên hostname
- `sshd`: tên dịch vụ
- `root`: user đích
- `10.10.3.100`: IP nguồn
- `8266`: port nguồn

Vậy thì tôi sẽ tạo ra một `decoder` cha lấy tên là `sshd` để định nghĩa dịch vụ **SSH** và `program_name` là `sshd` để xác định điều kiện của `decoder`, nghĩa là bất kỳ log nào có chứa `sshd` thì Wazuh sẽ hiểu và gán nó vào `decoder` này.

```python
<decoder name="sshd">
  <program_name>^sshd</program_name>
</decoder>
```

Tiếp theo, tôi sẽ tạo thêm các `decoder` con bên dưới để định nghĩa các trường có trong log.

Đầu tiên phải có `parent` với tên của `decoder` cha là `sshd`. `prematch` có thể chứa giá trị thay đổi xuất hiện trong log. Ví dụ, sự kiện login thành công thì log của nó sẽ chứa "Accepted" hoặc "Success". 

Với `regex`, Wazuh *decoders* sử dụng các [biểu thức chính quy](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html) để trích xuất các giá trị thay đổi và `order` sẽ gán các giá trị đó với các trường mà chúng ta muốn định nghĩa 

```python
<decoder name="sshd-success">
  <parent>sshd</parent>
  <prematch>^Accepted</prematch>
  <regex offset="after_prematch">^ \S+ for (\S+) from (\S+) port (\S+)</regex>
  <order>user, srcip, srcport</order>
  <fts>name, user, location</fts>
</decoder>
```

Ví dụ về một *decoder* của một log khác cũng thuộc `sshd`

```python
2020-03-25 08:23:20.933154-0700  localhost sshd[9265]: Connection reset by authenticating user user 192.168.33.1 port 51772 [preauth]
```

*Decoder* sẽ được viết như nhau

```python
<decoder name="sshd-reset">
  <parent>sshd</parent>
  <prematch>Connection reset</prematch>
  <regex offset="after_prematch">(\S+) (\S+) port (\d+)</regex>
  <order>user, srcip, srcport</order>
</decoder>
```

### Cú pháp biểu thức chính quy

***[Regular expressions - Biểu thức chính quy](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html)*** là tập hợp các mẫu dùng để tìm kiếm các bộ kí tự được kết hợp với nhau trong các chuỗi kí tự. Wazuh cũng sử dụng những `regex` đã rất phổ biến để xác định các mẫu.

Đọc thêm và thực hành về `regex` ở [đây](https://regex101.com/)

Trong documentation của Wazuh có sử dụng 3 loại `regex`. Tôi sẽ chỉ tổng hợp lại những gì tôi hay dùng

| Expressions | Valid characters |
| ------ | ------ |
| \w | A-Z, a-z, 0-9, -, @, _ |
| \d | 0-9 |
| \s | Spaces " " |
| \t | Tabs |
| \S | Mọi ký tự không thuộc \s |
| \\. | Tất cả |

**Modifiers**

| Expressions | Action |
| ------ | ------ |
| + | Khớp với 1 hoặc nhiều lần |
| * | Khớp với 0 hoặc nhiều lần |

**Special characters**

| Expressions | Action |
| ------ | ------ |
| ^ | Xác định phần đầu của văn bản |
| $ | Xác định phần cuối của văn bản |
| \| | Tạo một logic hoặc ngăn cách giữa 2 mẫu |

**Characters escaping**

| $ | ( | ) | \\ | \| | < |
| ------ | ------ | ------ | ------ | ------ | ------ |
| \\$ | \\( | \\) | \\\\ | \\\| | \\< |


### Ví dụ thực tế

Tôi sẽ lấy ví dụ về một log thực tế trong quá trình làm việc của mình (tất nhiên là nội dung log đã được sửa đổi)

```python
CEF:0|Imperva Inc.|SecureSphere|10.6.0|Recommended Signatures Policy for Web Applications|Recommended Signatures Policy for Web Applications|High|act=Block dst=10.11.12.13 dpt=443 duser=n/a src=103.187.191.141 spt=36614 proto=TCP rt=Apr 26 2023 13:08:12 cat=Alert cs1=Recommended Signatures Policy for Web Applications cs1Label=Policy cs2=mail.re-toor.vn cs2Label=ServerGroup cs3=mail cs3Label=ServiceName cs4=mail.re-toor.vn cs4Label=ApplicationName cs5===Nmap== scanner cs5Label=Description
```

Kiểm tra `decoder` với [wazuh-logtest](https://documentation.wazuh.com/current/user-manual/ruleset/testing.html?highlight=logtest#using-the-wazuh-dashboard-and-the-command-line-tool)

```python
root@dc-siemtest:/home/siemadmin# /var/ossec/bin/wazuh-logtest
Starting wazuh-logtest v4.4.5
Type one log per line

CEF:0|Imperva Inc.|SecureSphere|10.6.0|Recommended Signatures Policy for Web Applications|Recommended Signatures Policy for Web Applications|High|act=Block dst=10.11.12.13 dpt=443 duser=n/a src=103.187.191.141 spt=36614 proto=TCP rt=Apr 26 2023 13:08:12 cat=Alert cs1=Recommended Signatures Policy for Web Applications cs1Label=Policy cs2=mail.re-toor.vn cs2Label=ServerGroup cs3=mail cs3Label=ServiceName cs4=mail.re-toor.vn cs4Label=ApplicationName cs5=Nmap scanner cs5Label=Description

**Phase 1: Completed pre-decoding.
        full event: 'CEF:0|Imperva Inc.|SecureSphere|10.6.0|Recommended Signatures Policy for Web Applications|Recommended Signatures Policy for Web Applications|High|act=Block dst=10.11.12.13 dpt=443 duser=n/a src=103.187.191.141 spt=36614 proto=TCP rt=Apr 26 2023 13:08:12 cat=Alert cs1=Recommended Signatures Policy for Web Applications cs1Label=Policy cs2=mail.re-toor.vn cs2Label=ServerGroup cs3=mail cs3Label=ServiceName cs4=mail.re-toor.vn cs4Label=ApplicationName cs5=Nmap scanner cs5Label=Description'

**Phase 2: Completed decoding.
```

Phase 2 không có gì.

Phân tích qua về các phần có trong log này. Phần đầu tiên là các đoạn được phân tách với nhau bằng "\|", phần còn lại là các trường được phân tách bằng "=". Phần số 2 tôi đã có sẵn tên trường do log định nghĩa. Với phần đầu tiên thì có thể dựa vào nội dung của các đoạn để định nghĩa tên các trường. 

Đầu tiên tạo `decoder` cha với `prematch` 

```python
<decoder name="Imperva"> 
       <prematch>CEF:0\|Imperva Inc.\|</prematch> 
</decoder> 
```

Tiếp theo tạo `decoder` con định nghĩa các trường có trong phần 1. Không tính 2 đoạn đầu dùng để xác định loại log ở `prematch` thì tôi còn 5 đoạn phía sau để gán trường. Với mỗi trường tôi sẽ sử dụng 1 `regex` để đại diện cho giá trị có thể thay đổi.

```python
<decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>CEF:0\|Imperva Inc.\|(\.+)\|(\.+)\|(\.+)\|(\.+)\|(\.+)\|</regex> 
       <order>application,version,event.type,event.name,severity</order> 
</decoder>
```

Nếu chỉ có ký tự chữ thì có thể chọn `\S`, tuy nhiên "Imperva Inc." có 1 dấu chấm ở cuối nên tôi chọn `\.` đại diện cho ký tự bất kỳ. Tuy nhiên `\.` chỉ đại diện cho 1 ký tự, hiển thị được toàn bộ "Imperva Inc." tôi phải thêm `+` vào sau, `()` để nhóm nó thành 1 khối và `\` ở cuối để kết thúc khối này. 

Cuối cùng là `order` dùng để định nghĩa tên của các khối phía trên. Tôi có 5 khối cần gán tên, và phụ thuộc vào nội dung của khối đó để xác định khối này tên tên gì. Ví dụ như khối đầu tiên **SecureSphere** là tên của sản phẩm nên tôi sẽ gán tên khối này là `application`, hoặc khối cuối cùng nội dung là **High** - mức độ nghiêm trọng, tôi có thể gán tên `severity`.

Toàn bộ `decoder` của log trên có thể được viết như sau:

```python
<decoder name="Imperva"> 
       <prematch>CEF:0\|Imperva Inc.\|</prematch> 
    </decoder> 
 
    <!--<decoder name="Imperva_child"> -->
    <!--   <parent>Imperva</parent> -->
    <!--   <regex>\|SIEMintegration\|(\.+)\sfileId=</regex> -->
    <!--   <order>IncapRules</order> -->
    <!--</decoder> -->
    
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>CEF:0\|Imperva Inc.\|(\.+)\|(\.+)\|(\.+)\|(\.+)\|(\.+)\|</regex> 
       <order>application,version,event.type,event.name,severity</order> 
    </decoder>
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>fileId=(\.+) \w+=</regex> 
       <order>fileID</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>sourceServiceName=(\.+) \w+=</regex> 
       <order>sourceServiceName</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>siteid=(\.+) \w+=</regex> 
       <order>siteid</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>requestClientApplication=(\.+) \w+=</regex> 
       <order>requestClientApplication</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>deviceFacility=(\.+) \w+=</regex> 
       <order>deviceFacility</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>cs2=(\.+) \w+=</regex> 
       <order>cs2</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>cs2Label=(\.+) \w+=</regex> 
       <order>cs2Label</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>cs3=(\.+) \w+=</regex> 
       <order>cs3</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>cs3Label=(\.+) \w+=</regex> 
       <order>cs3Label</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>cs1=(\.+) \w+=</regex> 
       <order>cs1</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>cs1Label=(\.+) \w+=</regex> 
       <order>cs1Label</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>cs4=(\.+) \w+=</regex> 
       <order>cs4</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>cs4Label=(\.+) \w+=</regex> 
       <order>cs4Label</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>cs5=(\.+) \w+=</regex> 
       <order>cs5</order> 
    </decoder> 
    
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>cs5=(\.+)  \w+=</regex> 
       <order>cs5</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>cs5Label=(\.+) \w+=</regex> 
       <order>cs5Label</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>cs6(\.+) \w+=</regex> 
       <order>cs6</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>cs6Label=(\.+) \w+=</regex> 
       <order>cs6Label</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>cs7=(\.+) \w+=</regex> 
       <order>cs7</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>cs7Label=(\.+) \w+=</regex> 
       <order>cs7Label</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>cs8=(\.+) \w+=</regex> 
       <order>cs8</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>cs8Label=(\.+) \w+=</regex> 
       <order>cs8Label</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>customer=(\.+) \w+=</regex> 
       <order>customer</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>start=(\.+) \w+=</regex> 
       <order>start</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>request=(\.+) \w+=</regex> 
       <order>request</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>requestMethod=(\.+) \w+=</regex> 
       <order>requestMethod</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>app=(\.+) \w+=</regex> 
       <order>app</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>act=(\.+) \w+=</regex> 
       <order>action</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>deviceExternalId=(\.+) \w+=</regex> 
       <order>deviceExternalId</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>cpt=(\.+) \w+=</regex> 
       <order>cpt</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>src=(\.+) \w+=</regex> 
       <order>src</order> 
    </decoder> 
    
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>dst=(\.+) \w+=</regex> 
       <order>dst</order> 
    </decoder> 
    
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>dpt=(\.+) \w+=</regex> 
       <order>dpt</order> 
    </decoder> 
    
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>duser=(\.+) \w+=</regex> 
       <order>duser</order> 
    </decoder> 
    
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>rt=(\.+) \w+=</regex> 
       <order>rt</order> 
    </decoder> 
    
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>cat=(\.+) \w+=</regex> 
       <order>cat</order> 
    </decoder> 
    
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>proto=(\.+) \w+=</regex> 
       <order>protocol</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>ver=(\.+) \w+=</regex> 
       <order>ver</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>end=(\.+) \w+=</regex> 
       <order>end</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>fileType=(\.+) \w+=</regex> 
       <order>fileType</order> 
    </decoder> 
 
    <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>filePermission=(\.+) \w+=</regex> 
       <order>filePermission</order> 
    </decoder> 
 
   <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>cs9=(\.+) \w+=</regex> 
       <order>cs9</order> 
    </decoder> 
 
   <decoder name="Imperva_child"> 
       <parent>Imperva</parent> 
       <regex>cs9Label=(\.+) \w+</regex> 
       <order>cs9Label</order> 
    </decoder>
```

Kiểm tra `decoder` với [wazuh-logtest](https://documentation.wazuh.com/current/user-manual/ruleset/testing.html?highlight=logtest#using-the-wazuh-dashboard-and-the-command-line-tool)

```python
root@dc-siemtest:/home/siemadmin# /var/ossec/bin/wazuh-logtest
Starting wazuh-logtest v4.4.5
Type one log per line

CEF:0|Imperva Inc.|SecureSphere|10.6.0|Recommended Signatures Policy for Web Applications|Recommended Signatures Policy for Web Applications|High|act=Block dst=10.11.12.13 dpt=443 duser=n/a src=103.187.191.141 spt=36614 proto=TCP rt=Apr 26 2023 13:08:12 cat=Alert cs1=Recommended Signatures Policy for Web Applications cs1Label=Policy cs2=mail.re-toor.vn cs2Label=ServerGroup cs3=mail cs3Label=ServiceName cs4=mail.re-toor.vn cs4Label=ApplicationName cs5=Nmap scanner cs5Label=Description

**Phase 1: Completed pre-decoding.
        full event: 'CEF:0|Imperva Inc.|SecureSphere|10.6.0|Recommended Signatures Policy for Web Applications|Recommended Signatures Policy for Web Applications|High|act=Block dst=10.11.12.13 dpt=443 duser=n/a src=103.187.191.141 spt=36614 proto=TCP rt=Apr 26 2023 13:08:12 cat=Alert cs1=Recommended Signatures Policy for Web Applications cs1Label=Policy cs2=mail.re-toor.vn cs2Label=ServerGroup cs3=mail cs3Label=ServiceName cs4=mail.re-toor.vn cs4Label=ApplicationName cs5=Nmap scanner cs5Label=Description'

**Phase 2: Completed decoding.
        name: 'Imperva'
        action: 'Block'
        application: 'SecureSphere'
        cat: 'Alert'
        cs1: 'Recommended Signatures Policy for Web Applications'
        cs1Label: 'Policy'
        cs2: 'mail.re-toor.vn'
        cs2Label: 'ServerGroup'
        cs3: 'mail'
        cs3Label: 'ServiceName'
        cs4: 'mail.re-toor.vn'
        cs4Label: 'ApplicationName'
        cs5: 'Nmap scanner'
        dpt: '443'
        dst: '10.11.12.13'
        duser: 'n/a'
        event.name: 'Recommended Signatures Policy for Web Applications'
        event.type: 'Recommended Signatures Policy for Web Applications'
        protocol: 'TCP'
        rt: 'Apr 26 2023 13:08:12'
        severity: 'High'
        src: '103.187.191.141'
        version: '10.6.0'
```

Thành công.
