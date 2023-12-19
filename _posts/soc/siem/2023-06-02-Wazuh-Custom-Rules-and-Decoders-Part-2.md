---
layout: post
author: "Neo"
title: "Viết Decoders và Rules cho hệ thống Wazuh - Part 2"
date: "2023-06-02"
tags: siem
categories: soc
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

## Ruleset

### Là gì? Để làm gì?

Sau khi đã biên dịch log để Wazuh có thể hiểu được log của chúng ta đang "nói" gì, việc tiếp theo là làm cho nó hiện lên màn hình (dashboard) của chúng ta. Không phải tất cả các log Wazuh nhận được sẽ đều hiển thị lên màn hình, việc này sẽ gây loãng thông tin, làm cho quản trị viên mất rất nhiều thời gian để quản lý cũng như xác định đúng hiểm họa đang xảy ra đối với tổ chức của mình.

Việc viết rule chính là để "nói" cho Wazuh biết log nào cần hiển thị, hiển thị nó như thế nào và quan trọng nhất là dựa vào ***decoder*** nào.

Cũng giống như `decoder` con phụ thuộc vào `decoder` cha, `rule` khi được viết ra cũng phải dựa trên `decoder` cha. Vì Wazuh "làm việc" với `decoder` chứ không phải log, vậy nên nếu không theo `decoder` nào thì `rule` đó không có nội dung và cũng không có ý nghĩa.

Giống như **[Decoder](https://re-toor.me/soc/Wazuh-Custom-Rules-and-Decoders-Part-1.html)**, ***Rules*** cũng có những tùy chọn để định cấu hình. [Documentation của Wazuh](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/rules.html) đã định nghĩa tất cả các tùy chọn có thể được sử dụng trong khi viết rules. Tuy nhiên tôi cũng sẽ liệt kê những gì tôi hay dùng nhất. 

| Option | Values | Description |
| ------ | ------ | ----------- |
| [rule](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/rules.html#rules-rule) | [bảng](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/rules.html#rules-rule) | Tạo rule mới và các tùy chọn bên trong nó |
| [match](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/rules.html#rules-match) | [regex](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html#os-regex-syntax) | Kiểm tra xem trong log có chứa ký tự (sử dụng [sregex](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html#sregex-os-match-syntax)) trùng với ký tự được xác định hay không |
| [regex](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/rules.html#regex-rules) | [regex](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html#os-regex-syntax),[sregex](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html#sregex-os-match-syntax) hoặc [pcre2](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html#pcre2-syntax) | Giống với `match`, nhưng sử dụng [regex](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html#os-regex-syntax) |
| [decoded_as](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/rules.html#decoded-as) | Tên của decoder cha | Khớp với `decoder` cha |
| [field](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/rules.html#rules-field) | [regex](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/regex.html) | So sánh một trường trong phần [order](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/decoders.html#decoders-order) đã được định nghĩa trong `decoder` |
| [description](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/rules.html#description) | Ký tự bất kỳ | Mô tả về nội dung cảnh báo |
| [group](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/rules.html#group) | Ký tự bất kỳ | Phân loại cảnh báo theo nhóm |
| [if_sid](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/rules.html#if-sid) | Danh sách ID được phân tách bằng dấu phẩy | Giống như rule con phụ thuộc vào rule cha |

Các tùy chọn khác có thể được dùng trong các trường hợp nhất định tùy thuộc vào đặc điểm nội dung của các loại log có trong hệ thống.

### Viết như thế nào?

Tương tự như `decoder`, các `rules` cũng được viết dưới dạng XML, và như có đề cập ở trên, `rule` được viết dựa trên các trường của `decoder`, ngoài ra chúng ta cũng có thể dựa vào nội dung của log làm điều kiện để viết `rule` một cách chi tiết nhất.

Các rule này không nhất thiết phải sinh theo dạng cha-con, chúng có thể được tạo độc lập với nhau và đại diện cho một đặc điểm nào đó xuất hiện trong `decoder`.

Tôi sẽ quay lại ví dụ như đã làm ở `decoder` cho có tính thống nhất.

```python
CEF:0|Imperva Inc.|SecureSphere|10.6.0|Recommended Signatures Policy for Web Applications|Recommended Signatures Policy for Web Applications|High|act=Block dst=10.11.12.13 dpt=443 duser=n/a src=103.187.191.141 spt=36614 proto=TCP rt=Apr 26 2023 13:08:12 cat=Alert cs1=Recommended Signatures Policy for Web Applications cs1Label=Policy cs2=mail.re-toor.vn cs2Label=ServerGroup cs3=mail cs3Label=ServiceName cs4=mail.re-toor.vn cs4Label=ApplicationName cs5=Nmap scanner cs5Label=Description
```

Đầu tiên tôi sẽ cho nó vào 1 group nhằm mục đích tập hợp nhóm các loại rule dựa trên log có cùng đặc điểm chung. Ví dụ như log trên nhận được dưới dạng CEF (Common Event Format) và đây là log của Imperva, nên tôi sẽ cho các rule này vào nhóm Imperva và CEF

```python
<group name="Imperva,CEF,">

</group>
```

Bây giờ các rule về Imperva sẽ được viết bên trong group này.

Mặc dù `rule` không cần thiết phải viết dưới dạng cha-con, nhưng để cho rõ ràng và có hệ thống, tôi vẫn sẽ tạo rule đầu tiên chứa `decoder` mà rule sẽ phụ thuộc và thêm mô tả cho nó, coi nó giống như `rule` cha. Tất cả các `rule` tiếp theo chỉ cần phụ thuộc vào cha thì sẽ không cần khai báo lại `decoder` nữa.

```python
    <rule id="82002" level="0">
        <decoded_as>Imperva</decoded_as>
        <description>WAF messages grouped.</description>
    </rule>
```

Mỗi rule khi sinh ra đều phải có `id` và level - mức độ nghiêm trọng. Level sẽ quyết định xem rule nào được coi là cảnh báo (alert) xuất hiện trên dashboard còn rule nào thì không. Tiếp theo, `<decoder_as>` là trường dùng dể xác định rule này dựa tren `decoder` nào. Cuối cùng là `<description>` là mô tả của `rule`, trên dashboard của tôi sẽ sẽ hiện mô tả này.

[Cấu hình mặc định của Wazuh](https://documentation.wazuh.com/current/user-manual/manager/alert-threshold.html) cho chúng ta cảnh báo từ level 3, nghĩa là các log thuộc level 0 - 2 sẽ không được coi là alert và không hiện trên dashboard. Tôi sẽ cũng để theo cấu hình mặc định của Wazuh. Thực ra việc để cấu hình level bao nhiêu không quan trọng vì tôi có thể sửa được level của rule theo mức độ nghiêm trọng và theo cấu hình của Wazuh.

Bây giờ tôi sẽ xác định xem log này có những gì để có thể là điều kiện để viết rule cảnh báo. Để ý 1 chút thì tôi có trường `cs5Label` có giá trị là *Description*, có thể suy ra trường này là miêu tả cho trường `cs5`. `cs5` có giá trị là *Nmap scanner*, vậy thì trường có thể là đại diện cho nội dung của log này, tôi có thể dựa vào `cs5` làm điều kiện phụ thuộc để viết rule.

```python
    <rule id="82023" level="12">
        <if_sid>82002</if_sid>
        <match>Nmap scanner</match>
        <description>$(cs5)</description>
        <mitre>
            <id>T1046</id>
        </mitre>
    </rule>
```

Tôi đặt id của rule này là 82023. Với level 12 thì rule này sẽ sinh ra alert ở mức Critical - nghiêm trọng và sẽ báo đỏ trên dashboard, nhằm thông báo cho quản trị viên biết hệ thống đang bị rò quét port.

Với `<if_sid>` 82002 nghĩa là rule 82023 này sẽ phụ thuộc vào rule 82002, nếu log không được phát hiện bởi 82002 thì 82023 cũng sẽ không được kích hoạt.

Trường `<match>` dùng để xác định ký tự tồn tại trong log. Nếu như trong log có xuất hiện text "Nmap scanner" thì rule này sẽ được kích hoạt.

`<description>` sử dụng `$(cs5)` chỉ định rằng nếu rule này được kích hoạt thì mô tả của cảnh báo này sẽ xuất hiện với nội dung của trường `cs5`

Còn `<mitre>` là trường dùng để gắn các kỹ thuật được định nghĩa tại [MITRE ATT&CK®](https://attack.mitre.org/). Để gắn được giá trị này cần có hiểu biết về nó. Tôi sẽ viết về phần này trong 1 bài riêng.

Cuối cùng tôi có rule cơ bản như sau:

```python
<group name="Imperva,CEF,">

    <rule id="82002" level="0">
        <decoded_as>Imperva</decoded_as>
        <description>WAF messages grouped.</description>
    </rule>
    <rule id="82023" level="12">
        <if_sid>82002</if_sid>
        <match>Nmap scanner</match>
        <description>$(cs5)</description>
        <mitre>
            <id>T1046</id>
        </mitre>
    </rule>
</group>
```

Kiểm tra `rule` với [wazuh-logtest](https://documentation.wazuh.com/current/user-manual/ruleset/testing.html?highlight=logtest#using-the-wazuh-dashboard-and-the-command-line-tool)

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

**Phase 3: Completed filtering (rules).
        id: '82023'
        level: '12'
        description: 'Nmap scanner'
        groups: '['Imperva', 'CEF']'
        firedtimes: '1'
        mail: 'True'
        mitre.id: '['T1046']'
        mitre.tactic: '['Discovery']'
        mitre.technique: '['Network Service Discovery']'
**Alert to be generated.
```

Như vậy là rule đã được tạo thành công. 

## Phần kết

Với Wazuh, `decoder` và `rule` đều được chia làm 2 phần là mặc định và tùy chỉnh. 

| Option | Path |
| ------ | ------ |
| Default | /var/ossec/ruleset |
| Custom | /var/ossec/etc |

Phần mặc định (default) là các `decoders` và `rules` được Wazuh viết sẵn, không thể thay đổi hay ghi đè các file này ở trên dashboard. Và cho dù tôi có cố gắng thay đổi nó bằng việc chỉnh sửa file trong server, thì khi cập nhật các phiên bản mới của Wazuh, mọi thay đổi của tôi cũng sẽ quay về mặc định. Tôi chỉ có thể tạo rule mới và ghi đề vào rule cũ với lệnh [overwrite](https://documentation.wazuh.com/current/user-manual/ruleset/custom.html#changing-an-existing-rule)

Phần custom là các `decoders` và `rule` được người quản trị tự tạo.

> Lưu ý: `decoders` và `rules` chỉ có thể được kích hoạt nếu nằm đúng vị trí trong các thư mục phía trên.

Mỗi rule và decoder khi được tạo mới hay có bất kỳ thay đổi gì đều phải restart lại Wazuh-manager. Tôi đã tạo và cập nhật custom `decoders` - `rules` của mình ở [đây](https://github.com/re-toor/Wazuh-Rules).
