---
layout: post
author: "Neo"
title: "Cấu hình Wazuh vulnerability detection"
date: "2022-05-09"
tags: siem
categories: soc
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

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

Vì là offile update nên Wazuh sẽ không tự động cập nhật các pack trên mỗi khi có CVE hay lỗ hổng mới. Tôi sẽ sử dụng script để nó update tự động, và ở đây tôi chọn update tự động mỗi 1h.

### Red Hat Security Data JSON feed



