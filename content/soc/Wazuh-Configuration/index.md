---
title: "Cấu hình Wazuh vulnerability detection"
date: "2023-05-09"
tags: 
  - siem
---

Sau khi đã cài đặt xong các thành phần cũng như cài đặt những Agent đầu tiên trong hệ thống để bắt đầu nhận log, tôi nhận ra một vấn đề là Wazuh manager của tôi không đưa ra được cảnh bảo, không phát hiện được CVE hay điểm yếu nào. Chắc chắn không phải do máy tôi sạch, làm gì có máy nào "sạch" hoàn toàn =)))

Sau một lúc đọc Doc của Wazuh và đối chiếu với hệ thống, tôi nhận ra tôi đã quên mất trong hệ thống của mình có web gateway, và việc các trang web bị chặn bởi proxy là bình thường. Vậy nên việc của tôi bây h là cần phải cấu hình thủ công để wazuh tự động cập nhật mỗi khi có lỗ hổng mới.

Quy trình của việc này sẽ là tải các file xml chứa thông tin của các CVE tương ứng với các phiên bản Agent và sau đó tạo 1 cron job để nó chạy tự động. 

Bài viết này dựa trên phần [Offline-update](https://documentation.wazuh.com/current/user-manual/capabilities/vulnerability-detection/offline-update.html) của Wazuh documentation.

## Downloading the specific vulnerability files

Việc đầu tiên là tải các file xml chứa lỗ hổng cho từng phiên bản hệ điều hành của Agent có thể xuất hiện trong môi trường doanh nghiệp của chúng ta.

Như đã nói ở trên, do không thể tải trực tiếp các file từ máy chủ wazuh nên tôi sẽ tải tất cả về máy cá nhân và chuyển nó về máy chủ wazuh (có thể bằng ssh hoặc chức năng truyền file trên các phần mềm ssh như: Mobaxterm, Termius, ...)

Trước tiên tôi sẽ tạo 1 thư mục để lưu hết các file sẽ tải về.

```python
mkdir /var/ossec/offline-update
```

### Linux OS

Tải xuống các file của các phiên bản Linux theo link bên dưới.

#### Ubuntu

| OS | Files |
| ------ | ------ |
| Trusty | [com.ubuntu.trusty.cve.oval.xml.bz2](https://security-metadata.canonical.com/oval/com.ubuntu.trusty.cve.oval.xml.bz2) |
| Xenial | [com.ubuntu.xenial.cve.oval.xml.bz2](https://security-metadata.canonical.com/oval/com.ubuntu.xenial.cve.oval.xml.bz2) |
| Bionic | [com.ubuntu.bionic.cve.oval.xml.bz2](https://security-metadata.canonical.com/oval/com.ubuntu.bionic.cve.oval.xml.bz2) |
| Focal | [com.ubuntu.focal.cve.oval.xml.bz2](https://security-metadata.canonical.com/oval/com.ubuntu.focal.cve.oval.xml.bz2) |
| Jammy | [com.ubuntu.jammy.cve.oval.xml.bz2](https://security-metadata.canonical.com/oval/com.ubuntu.jammy.cve.oval.xml.bz2) |


#### Debian

| OS | Files |
| ------ | ------ |
| Bullseye | [oval-definitions-bullseye.xml](https://www.debian.org/security/oval/oval-definitions-bullseye.xml) |
| Buster | [oval-definitions-buster.xml](https://www.debian.org/security/oval/oval-definitions-buster.xml) |

#### Debian Security Tracker JSON feed

| OS | Files |
| ------ | ------ |
| ALL | [Debian Security Tracker JSON](https://security-tracker.debian.org/tracker/data/json) |

#### Red Hat

| OS | Files |
| ------ | ------ |
| 5 | [com.redhat.rhsa-RHEL5.xml.bz2](https://www.redhat.com/security/data/oval/com.redhat.rhsa-RHEL5.xml.bz2)|
| 6 | [rhel-6-including-unpatched.oval.xml.bz2](https://www.redhat.com/security/data/oval/v2/RHEL6/rhel-6-including-unpatched.oval.xml.bz2) |
| 7 | [rhel-7-including-unpatched.oval.xml.bz2](https://www.redhat.com/security/data/oval/v2/RHEL7/rhel-7-including-unpatched.oval.xml.bz2) |
| 8 | [rhel-8-including-unpatched.oval.xml.bz2](https://www.redhat.com/security/data/oval/v2/RHEL8/rhel-8-including-unpatched.oval.xml.bz2) |
| 9 | [rhel-9-including-unpatched.oval.xml.bz2](https://www.redhat.com/security/data/oval/v2/RHEL9/rhel-9-including-unpatched.oval.xml.bz2) |

### Windows OS

| OS | Files |
| ------ | ------ |
| ALL | [msu-updates.json.gz](https://feed.wazuh.com/vulnerability-detector/windows/msu-updates.json.gz) |

Sau khi đã tải xong tất cả các file trên, tôi sẽ phải chuyển nó về thư mục */offline-update/* mà tôi đã tạo từ ban đầu trên máy chủ wazuh.

## Offline update script

Vì là offile update nên Wazuh sẽ không tự động cập nhật các pack trên mỗi khi có CVE hay lỗ hổng mới. Tôi sẽ sử dụng script để nó update tự động, và ở đây tôi chọn update tự động sau mỗi 1h.

### Red Hat Security Data JSON feed

Wazuh có cung cấp 2 script để phục vụ cho việc update thủ công. [Script](https://raw.githubusercontent.com/wazuh/wazuh/master/tools/vulnerability-detector/rh-generator.sh) đầu tiền dùng để update các CVE mới mỗi khi nó được công bố, và được cập nhật liên tục từ năm 1999. 

Đầu tiền tôi tạo 1 folder để lưu các feeds

```python
mkdir /var/ossec/offline-update/rh-feed
```

Tải script đó về và chạy 

```python
/var/ossec/offline-update/rh-generator.sh /var/ossec/offline-update/rh-feed
```

Sau khi chạy script đã chạy xong, vào lại thư mục để kiểm tra

```python
root@siem:~# ll /var/ossec/offline-update/rh-feed/
total 22312
drwxr-xr-x 2 root wazuh    4096 Jun  2 16:00 ./
drwxr-xr-x 4 root wazuh    4096 Apr 20 16:44 ../
-rw-r--r-- 1 root root   775209 Jun  2 16:00 redhat-feed10.json
-rw-r--r-- 1 root root   956401 Jun  2 16:00 redhat-feed11.json
-rw-r--r-- 1 root root   966884 Jun  2 16:00 redhat-feed12.json
-rw-r--r-- 1 root root   730036 Jun  2 16:00 redhat-feed13.json
-rw-r--r-- 1 root root   686898 Jun  2 16:00 redhat-feed14.json
-rw-r--r-- 1 root root   880443 Jun  2 16:00 redhat-feed15.json
-rw-r--r-- 1 root root   727703 Jun  2 16:00 redhat-feed16.json
-rw-r--r-- 1 root root   752990 Jun  2 16:00 redhat-feed17.json
-rw-r--r-- 1 root root   869173 Jun  2 16:00 redhat-feed18.json
-rw-r--r-- 1 root root   946425 Jun  2 16:00 redhat-feed19.json
-rw-r--r-- 1 root root   666265 Jun  2 16:00 redhat-feed1.json
-rw-r--r-- 1 root root   991033 Jun  2 16:00 redhat-feed20.json
-rw-r--r-- 1 root root   934536 Jun  2 16:00 redhat-feed21.json
-rw-r--r-- 1 root root   814900 Jun  2 16:00 redhat-feed22.json
-rw-r--r-- 1 root root   776735 Jun  2 16:00 redhat-feed23.json
-rw-r--r-- 1 root root   572706 Jun  2 16:00 redhat-feed24.json
-rw-r--r-- 1 root root   567478 Jun  2 16:00 redhat-feed25.json
-rw-r--r-- 1 root root   518678 Jun  2 16:00 redhat-feed26.json
-rw-r--r-- 1 root root   511669 Jun  2 16:00 redhat-feed27.json
-rw-r--r-- 1 root root   399497 Jun  2 16:00 redhat-feed28.json
-rw-r--r-- 1 root root   137052 Jun  2 16:00 redhat-feed29.json
-rw-r--r-- 1 root root  1026736 Jun  2 16:00 redhat-feed2.json
-rw-r--r-- 1 root root   898897 Jun  2 16:00 redhat-feed3.json
-rw-r--r-- 1 root root   797044 Jun  2 16:00 redhat-feed4.json
-rw-r--r-- 1 root root   803644 Jun  2 16:00 redhat-feed5.json
-rw-r--r-- 1 root root   913641 Jun  2 16:00 redhat-feed6.json
-rw-r--r-- 1 root root  1422008 Jun  2 16:00 redhat-feed7.json
-rw-r--r-- 1 root root   853773 Jun  2 16:00 redhat-feed8.json
-rw-r--r-- 1 root root   884240 Jun  2 16:00 redhat-feed9.json
```

### National Vulnerability Database

[Script](https://raw.githubusercontent.com/wazuh/wazuh/master/tools/vulnerability-detector/nvd-generator.sh) thứ 2 dùng để update thủ công cơ sở dữ liệu lỗ hổng NVD. Tôi cũng phải tạo 1 thư mục để cập nhật lỗ hổng của các năm cho đến thời điểm hiện tại.

```python
mkdir /var/ossec/offline-update/nvd-feed
```

Để chạy script này, tôi cần xác định năm mà tôi muốn lấy lỗ hổng. Tôi sẽ lấy tất cả lỗ hổng và cập nhật nó kể từ năm 2002 đến nay

```python
/var/ossec/offline-update/nvd-generator.sh 2002 /var/ossec/offline-update/nvd-feed
```

Sau khi chạy script đã chạy xong, vào lại thư mục để kiểm tra

```python
root@siem:~# ll /var/ossec/offline-update/nvd-feed/
total 58420
drwxr-xr-x 2 root wazuh    4096 Jun  1 18:00 ./
drwxr-xr-x 4 root wazuh    4096 Apr 20 16:44 ../
-rw-r--r-- 1 root root  1460135 Jun  2 15:00 nvd-feed2002.json.gz
-rw-r--r-- 1 root root   435532 Jun  2 15:00 nvd-feed2003.json.gz
-rw-r--r-- 1 root root    97973 Jun  2 15:03 nvd-feed2004.json.gz
-rw-r--r-- 1 root root  1345598 Jun  2 15:03 nvd-feed2005.json.gz
-rw-r--r-- 1 root root  2131988 Jun  2 15:03 nvd-feed2006.json.gz
-rw-r--r-- 1 root root  2108354 Jun  2 15:03 nvd-feed2007.json.gz
-rw-r--r-- 1 root root  2160683 Jun  2 15:03 nvd-feed2008.json.gz
-rw-r--r-- 1 root root  1965308 Jun  2 15:03 nvd-feed2009.json.gz
-rw-r--r-- 1 root root  1925207 Jun  2 15:03 nvd-feed2010.json.gz
-rw-r--r-- 1 root root  1813239 Jun  2 15:03 nvd-feed2011.json.gz
-rw-r--r-- 1 root root  2021518 Jun  2 15:03 nvd-feed2012.json.gz
-rw-r--r-- 1 root root  2415597 Jun  2 15:03 nvd-feed2013.json.gz
-rw-r--r-- 1 root root  2345483 Jun  2 15:03 nvd-feed2014.json.gz
-rw-r--r-- 1 root root  2239999 Jun  2 15:04 nvd-feed2015.json.gz
-rw-r--r-- 1 root root  2701974 Jun  2 15:04 nvd-feed2016.json.gz
-rw-r--r-- 1 root root  3832210 Jun  2 15:04 nvd-feed2017.json.gz
-rw-r--r-- 1 root root  4138080 Jun  2 15:04 nvd-feed2018.json.gz
-rw-r--r-- 1 root root  4706272 Jun  2 15:04 nvd-feed2019.json.gz
-rw-r--r-- 1 root root  5726147 Jun  2 15:04 nvd-feed2020.json.gz
-rw-r--r-- 1 root root  6358273 Jun  2 15:04 nvd-feed2021.json.gz
-rw-r--r-- 1 root root  6028683 Jun  2 15:04 nvd-feed2022.json.gz
-rw-r--r-- 1 root root  1819248 Jun  2 15:04 nvd-feed2023.json.gz
```

### Automate cronjob

Tôi sẽ tạo 1 cronjob để sau mỗi giờ, hệ thống có thể tự chạy 2 script và tự động update.

```python
nano /etc/crontab
```

Thêm 2 dòng bên dưới vào cuối file crontab

```python
0 * * * * root /var/ossec/offline-update/rh-generator.sh /var/ossec/offline-update/rh-feed/
0 * * * * root /var/ossec/offline-update/nvd-generator.sh 2002 /var/ossec/offline-update/nvd-feed/
```

Lưu lại file và khởi động lại dịch vụ cron

```python
systemctl restart cron
```

## Offline update sample

Sau khi đã thực hiện các bước tải và định vị trí cho các file, tôi sẽ cấu hình lại đường dẫn trong file *__ossec.conf__* giống như bên dưới

```python
  <vulnerability-detector>
    <enabled>yes</enabled>
    <interval>5m</interval>
    <min_full_scan_interval>6h</min_full_scan_interval>
    <run_on_start>yes</run_on_start>

    <!-- Ubuntu OS vulnerabilities -->
    <provider name="canonical">
      <enabled>yes</enabled>
      <os path="/var/ossec/offline-update/com.ubuntu.trusty.cve.oval.xml.bz2">trusty</os>
      <os path="/var/ossec/offline-update/com.ubuntu.xenial.cve.oval.xml.bz2">xenial</os>
      <os path="/var/ossec/offline-update/com.ubuntu.bionic.cve.oval.xml.bz2">bionic</os>
      <os path="/var/ossec/offline-update/com.ubuntu.focal.cve.oval.xml.bz2">focal</os>
      <os path="/var/ossec/offline-update/com.ubuntu.jammy.cve.oval.xml.bz2">jammy</os>
      <update_interval>1h</update_interval>
    </provider>

    <!-- Debian OS vulnerabilities -->
    <provider name="debian">
      <enabled>yes</enabled>
      <os path="/var/ossec/offline-update/oval-definitions-buster.xml">buster</os>
      <os path="/var/ossec/offline-update/oval-definitions-bullseye.xml">bullseye</os>
      <path>/var/ossec/offline-update/security_tracker_local.json</path>
      <update_interval>1h</update_interval>
    </provider>

    <!-- RedHat OS vulnerabilities -->
    <provider name="redhat">
      <enabled>yes</enabled>
      <os path="/var/ossec/offline-update/com.redhat.rhsa-RHEL5.xml.bz2">5</os>
      <os path="/var/ossec/offline-update/rhel-6-including-unpatched.oval.xml.bz2">6</os>
      <os path="/var/ossec/offline-update/rhel-7-including-unpatched.oval.xml.bz2">7</os>
      <os path="/var/ossec/offline-update/rhel-8-including-unpatched.oval.xml.bz2">8</os>
      <os path="/var/ossec/offline-update/rhel-9-including-unpatched.oval.xml.bz2">9</os>
      <path>/var/ossec/offline-update/rh-feed/redhat-feed[[:digit:]]\+\.json$</path>
      <update_interval>1h</update_interval>
    </provider>

    <!-- Amazon Linux OS vulnerabilities -->
    <provider name="alas">
      <enabled>no</enabled>
      <os>amazon-linux</os>
      <os>amazon-linux-2</os>
      <update_interval>1h</update_interval>
    </provider>

    <!-- SUSE OS vulnerabilities -->
    <provider name="suse">
      <enabled>no</enabled>
      <os>11-server</os>
      <os>11-desktop</os>
      <os>12-server</os>
      <os>12-desktop</os>
      <os>15-server</os>
      <os>15-desktop</os>
      <update_interval>1h</update_interval>
    </provider>

    <!-- Arch OS vulnerabilities -->
    <provider name="arch">
      <enabled>no</enabled>
      <update_interval>1h</update_interval>
    </provider>

    <!-- Windows OS vulnerabilities -->
    <provider name="msu">
      <enabled>yes</enabled>
      <path>/var/ossec/offline-update/msu-updates\.json\.gz$</path>
      <update_interval>1h</update_interval>
    </provider>

    <!-- Aggregate vulnerabilities -->
    <provider name="nvd">
      <enabled>yes</enabled>
      <path>/var/ossec/offline-update/nvd-feed/nvd-feed[[:digit:]]\{4\}\.json\.gz$</path>
      <update_from_year>2010</update_from_year>
      <update_interval>1h</update_interval>
    </provider>

  </vulnerability-detector>
```

Xem thêm về cấu hình tại [đây](https://documentation.wazuh.com/current/user-manual/capabilities/vulnerability-detection/offline-update.html)




