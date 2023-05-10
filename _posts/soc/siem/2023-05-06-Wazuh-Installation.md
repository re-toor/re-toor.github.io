---
layout: post
author: "Neo"
title: "Cài đặt và cấu hình hệ thống WAZUH"
date: "2022-05-06"
tags: siem
categories: soc
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

![intro](/assets/img/2023-05-06-Wazuh-Installation/1.png)

Do đặc thù công việc nên từ một người làm Pentest tôi phải chuyển sang làm SOC và đảm nhiệm việc xây dựng hệ thống SIEM. Mặc dù thời điểm hiện tại có rất nhiều sản phẩm thương mại đã hoàn thiện, chỉ chờ doanh nghiệp mua về và vận hành thôi. Tuy nhiên chi phí cho các sản phẩm này là một trở ngại đối với các doanh nghiệp vừa và nhỏ (nó đắt :laughing:). 

Vậy nên vừa để có thêm kiến thức và để tiết kiệm chi phí, tôi sẽ tự xây dựng hệ thống SIEM dựa trên open source. Một trong những open source mạnh mẽ nhất (hoặc là do tôi nghĩ thế) là [WAZUH](https://documentation.wazuh.com/current/getting-started/index.html)

*Wazuh là một nền tảng bảo mật mã nguồn mở và miễn phí, hợp nhất các khả năng của XDR và ​​SIEM, không chỉ cho phép các công ty phát hiện các mối đe dọa tinh vi mà còn có thể giúp ngăn ngừa vi phạm và rò rỉ dữ liệu xảy ra.*

Hiện tại thì tôi đang triển khai nó trên môi trường thực tế, vậy nên chắc chắn là nó hoạt động.

## Kiến trúc

![kiến trúc](/assets/img/2023-05-06-Wazuh-Installation/2.png)

Đọc thêm về cấu trúc tại [đây](https://documentation.wazuh.com/current/getting-started/architecture.html) hoặc là [đây](https://github.com/hocchudong/ghichep-SOC/blob/master/ghichep-wazuh/ghichep-overview-wazuh.md)

*Hiện tại thì WAZUH đang ở phiên bản 4.4 vậy nên tất cả cài đặt và cấu hình trong bài này cũng sẽ ở phiên bản 4.4, trong trường hợp có các cập nhật mới, chỉ cần thay đổi tên phiên bản (nếu) xuất hiện trong các link cài đặt.*

## WAZUH INDEXER

*Để không gặp các lỗi trong quá trình cài đặt, phải sử dụng quyền root*

### Certificates creation

Tải tool tạo cert và file config của wazuh về

```python
curl -sO https://packages.wazuh.com/4.4/wazuh-certs-tool.sh
curl -sO https://packages.wazuh.com/4.4/config.yml
```

Vào `config.yml` 

```python
nano config.yml
```

Thay đổi các giá trị “name” và “ip” thành tên là ip của server (Trường hợp tách wazuh-indexer và wazuh-manager thành 2 máy chủ vật lý thì phải đặt đúng ip của từng máy chủ). 

```python
nodes:
  # Wazuh indexer nodes
  indexer:
    - name: dc-indexer
      ip: 10.10.24.1
    #- name: node-2
    #  ip: <indexer-node-ip>
    #- name: node-3
    #  ip: <indexer-node-ip>

  # Wazuh server nodes
  # If there is more than one Wazuh server
  # node, each one must have a node_type
  server:
    - name: dc-manager
      ip: 10.10.24.1
    #  node_type: master
    #- name: wazuh-2
    #  ip: <wazuh-manager-ip>
    #  node_type: worker
    #- name: wazuh-3
    #  ip: <wazuh-manager-ip>
    #  node_type: worker

  # Wazuh dashboard nodes
  dashboard:
    - name: dc-dashboard
      ip: 10.10.24.1
```

Chạy *wazuh-certs-tool.sh* để tạo cert

```python
bash ./wazuh-certs-tool.sh -A
tar -cvf ./wazuh-certificates.tar -C ./wazuh-certificates/ .
```

### Node installation

Cài đặt các gói còn thiếu

```python
apt-get install debconf adduser procps
apt-get install gnupg apt-transport-https
```

Tạo wazuh repository

```python
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
apt-get update
```

Cài đặt wazuh-indexer

```python
apt-get -y install wazuh-indexer
```

Sửa _/etc/wazuh-indexer/opensearch.yml_ với các thông số sau

```python
network.host: “10.10.24.1”
node.name: “dc-indexer”
cluster.initial_master_nodes:
- "dc-indexer"
….
discovery.seed_hosts:		# bỏ dấu “#” ở đầu dòng 
  - "10.10.24.1"			# bỏ dấu “#” ở đầu dòng
….
path.logs: /var/log/wazuh-indexer
bootstrap.memory_lock: true
….
plugins.security.nodes_dn:
- "CN=dc-indexer,OU=Wazuh,O=Wazuh,L=California,C=US"
#- "CN=node-2,OU=Wazuh,O=Wazuh,L=California,C=US"
```

Tạo cert. Đặt tên node name giống tên node name cấu hình trong file _opensearch.yml_

```python
NODE_NAME=dc-indexer
mkdir /etc/wazuh-indexer/certs
tar -xf ./wazuh-certificates.tar -C /etc/wazuh-indexer/certs/ ./$NODE_NAME.pem ./$NODE_NAME-key.pem ./admin.pem ./admin-key.pem ./root-ca.pem
mv -n /etc/wazuh-indexer/certs/$NODE_NAME.pem /etc/wazuh-indexer/certs/indexer.pem
mv -n /etc/wazuh-indexer/certs/$NODE_NAME-key.pem /etc/wazuh-indexer/certs/indexer-key.pem
chmod 500 /etc/wazuh-indexer/certs
chmod 400 /etc/wazuh-indexer/certs/*
chown -R wazuh-indexer:wazuh-indexer /etc/wazuh-indexer/certs
```

Chỉnh sửa giới hạn tài nguyên hệ thống

```python
nano /usr/lib/systemd/system/wazuh-indexer.service
```

```python
[Service]
LimitMEMLOCK=infinity	# Thêm dòng này 
```

Tăng kích thước không gian heap

```python
nano /etc/wazuh-indexer/jvm.options
```

Sửa 1 thành 4

```python
# Xms represents the initial size of total heap space
# Xmx represents the maximum size of total heap space

-Xms4g
-Xmx4g
```

Bật và chạy dịch vụ

```python
systemctl daemon-reload
systemctl enable wazuh-indexer
systemctl start wazuh-indexer
```

### Cluster initialization

Chạy file _/usr/share/wazuh-indexer/bin/indexer-security-init.sh_ trên các node để tải lên thông tin các cert đã tạo

```python
/usr/share/wazuh-indexer/bin/indexer-security-init.sh
```

Kiểm tra tất cả các cài đặt đã thành công và dịch vụ không có lỗi (Lưu ý: tắt proxy trước khi thực hiện lệnh curl)

```python
curl -k -u admin:admin https://10.10.24.1:9200
```

Kết quả giống bên dưới là thành công

```python
{
  "name" : "dc-indexer",
  "cluster_name" : "wazuh-cluster",
  "cluster_uuid" : "bMz0BKdlRVui5jF-mlt6yg",
  "version" : {
    "number" : "7.10.2",
    "build_type" : "rpm",
    "build_hash" : "f2f809ea280ffba217451da894a5899f1cec02ab",
    "build_date" : "2022-12-12T22:17:42.341124910Z",
    "build_snapshot" : false,
    "lucene_version" : "8.10.1",
    "minimum_wire_compatibility_version" : "7.10.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "The OpenSearch Project: https://opensearch.org/"
}
```

## WAZUH MANAGER

### Wazuh server node installation

- Cài đặt wazuh-manager

```python
apt-get -y install wazuh-manager
```

Chạy wazuh-manager

```python
systemctl daemon-reload
systemctl enable wazuh-manager
systemctl start wazuh-manager
```

Kiểm tra trạng thái wazuh-manager

```python
systemctl status wazuh-manager
```

- Cài đặt filebeat

```python
apt-get -y install filebeat
```

Tải file cấu hình filebeat

```python
curl -so /etc/filebeat/filebeat.yml https://packages.wazuh.com/4.4/tpl/wazuh/filebeat/filebeat.yml
```

Sửa cấu hình filebeat với hosts là ip của indexer node

```python
hosts: ["10.10.24.1:9200"]
```

Tạo keystore

```python
filebeat keystore create
echo admin | filebeat keystore add username --stdin --force
echo admin | filebeat keystore add password --stdin --force
```

Tải template cho indexer

```python
curl -so /etc/filebeat/wazuh-template.json https://raw.githubusercontent.com/wazuh/wazuh/4.4/extensions/elasticsearch/7.x/wazuh-template.json
chmod go+r /etc/filebeat/wazuh-template.json
```

Cài đặt các module của wazuh cho filebeat

```python
curl -s https://packages.wazuh.com/4.x/filebeat/wazuh-filebeat-0.2.tar.gz | tar -xvz -C /usr/share/filebeat/module
```

Tạo cert cho filebeat, đặt tên NODE_NAME giống với tên của server trong file *config.yml*

```python
NODE_NAME=dc-manager
mkdir /etc/filebeat/certs
tar -xf ./wazuh-certificates.tar -C /etc/filebeat/certs/ ./$NODE_NAME.pem ./$NODE_NAME-key.pem ./root-ca.pem
mv -n /etc/filebeat/certs/$NODE_NAME.pem /etc/filebeat/certs/filebeat.pem
mv -n /etc/filebeat/certs/$NODE_NAME-key.pem /etc/filebeat/certs/filebeat-key.pem
chmod 500 /etc/filebeat/certs
chmod 400 /etc/filebeat/certs/*
chown -R root:root /etc/filebeat/certs
```

Chạy filebeat

```python
systemctl daemon-reload
systemctl enable filebeat
systemctl start filebeat
```

Kiểm tra trạng thái filebeat

```python
filebeat test output
```

Kết quả giống bên dưới là thành công

```python
elasticsearch: https://10.10.24.1:9200...
  parse url... OK
  connection...
    parse host... OK
    dns lookup... OK
    addresses: 127.0.0.1
    dial up... OK
  TLS...
    security: server's certificate chain verification is enabled
    handshake... OK
    TLS version: TLSv1.3
    dial up... OK
  talk to server... OK
  version: 7.10.2
```

## WAZUH DASHBOARD

Tải các package còn thiếu

```python
apt-get install debhelper tar curl libcap2-bin
```

Tải wazuh-dashboard

```python
apt-get -y install wazuh-dashboard
```

Sửa cấu hình dashboard

```python
nano /etc/wazuh-dashboard/opensearch_dashboards.yml
```

Tạo cert cho dashboard, đặt tên NODE_NAME giống với tên của dashboard trong file config.yml

```python
NODE_NAME=dc-dashboard
mkdir /etc/wazuh-dashboard/certs
tar -xf ./wazuh-certificates.tar -C /etc/wazuh-dashboard/certs/ ./$NODE_NAME.pem ./$NODE_NAME-key.pem ./root-ca.pem
mv -n /etc/wazuh-dashboard/certs/$NODE_NAME.pem /etc/wazuh-dashboard/certs/dashboard.pem
mv -n /etc/wazuh-dashboard/certs/$NODE_NAME-key.pem /etc/wazuh-dashboard/certs/dashboard-key.pem
chmod 500 /etc/wazuh-dashboard/certs
chmod 400 /etc/wazuh-dashboard/certs/*
chown -R wazuh-dashboard:wazuh-dashboard /etc/wazuh-dashboard/certs
```

Chạy dashboard

```python
systemctl daemon-reload
systemctl enable wazuh-dashboard
systemctl start wazuh-dashboard
```

Truy cập vào dashboard trên web theo địa chỉ https://10.10.24.1 với username và password mặc định *admin:admin*

Đổi password mặc định của tài khoản admin

```python
curl -so wazuh-passwords-tool.sh https://packages.wazuh.com/4.4/wazuh-passwords-tool.sh
```

```python
bash wazuh-passwords-tool.sh -u admin -p <mật khẩu>
```

> _**Lưu ý: Nếu hệ thống có proxy (internet gateway) thì phải đặt proxy trước khi cài đặt**_
> 
> - Set proxy cho apt để tải các package trên trang chủ Ubuntu
> 	
> `nano /etc/apt/apt.conf`
> 		
> `Acquire::https::Proxy "http://ip:port";`
> 		
> `Acquire::http::Proxy "http://ip:port";`
> 		
> Lưu file và login lại vào account
> 	
> - Set proxy cho server
> 	
> `nano /etc/environment`
> 		
> Thêm vào file các dòng sau:
> 	
> `HTTP_PROXY="http://ip:port"`
> 		
> `HTTPS_PROXY="http://ip:port"`
> 		
> `NO_PROXY="10.10.24.1,localhost,127.0.0.1,::1"`
> 		
> Lưu file và login lại vào account
