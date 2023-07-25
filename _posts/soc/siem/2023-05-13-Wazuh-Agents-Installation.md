---
layout: post
author: "Neo"
title: "Cài đặt và cấu hình Wazuh-agents"
date: "2023-05-13"
tags: siem
categories: soc
---

0. this unordered seed list will be replaced by toc as unordered list
{:toc}

Wazuh cung cấp phần mềm phía người dùng để thu thập và giám sát các hoạt động của người dùng trên máy đó. Các Wazuh agents sẽ chạy ngầm giống như Windows Defenders (ví dụ thế), thu thập toàn bộ các thay đổi trên máy tính của người dùng và gửi log về Wazuh manager.

Tuy nhiên, để agent được giám sát một cách chi tiết và mạnh mẽ nhất, cần cài thêm công cụ hỗ trợ cho Wazuh-agent, đó là Sysmon (cho Windows) và Packetbeat (cho Linux)

## Windows OS

### Sysmon 

[*System Monitor (Sysmon)*](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon) là một dịch vụ hệ thống của Windows cho phép người dùng có thể theo dõi được các hoạt động diễn ra trong windows như việc tạo quy trình, các kết nối mạng hay sự thay đổi của 1 file nào đó. Vì vậy nó rất hữu ích trong việc phân tích và quản lý log trên SIEM.

Để cho việc chuẩn hóa log được dễ dàng hơn, tôi sẽ sử dụng một cấu hình Sysmon có sẵn mà tôi tìm thấy trên github ở [đây](https://github.com/olafhartong/sysmon-modular)

Trong một các môi trường doanh nghiệp, việc sử dụng proxy server sẽ ngăn chúng ta chạy các scripts tự động tải file trực tiếp từ internet. Vậy nên cách làm của tôi là tải về trước và chạy sau (áp dụng cho cả Windows và Linux).

Việc đầu tiên là tải Sysmon từ trang chủ của [Micorsoft](https://download.sysinternals.com/files/SysinternalsSuite.zip) và giải nén nó

Sau đó tải [file config](https://raw.githubusercontent.com/pcsg-community/sysmon-config/main/sysmon-pcsg-daena-default.xml) ở github repo phía trên về và lưu nó vào thư mục vừa giải nén phía trên.

Tiếp theo tôi sẽ tạo 1 file Powershell để chạy Sysmon với cấu hình của file config phía trên.

```python
$serviceName = 'Sysmon64'
If (Get-Service $serviceName -ErrorAction SilentlyContinue) {
write-host ('Sysmon Is Already Installed')
} else {
Invoke-Command {reg.exe ADD HKCU\Software\Sysinternals /v EulaAccepted /t REG_DWORD /d 1 /f}
Invoke-Command {reg.exe ADD HKU\.DEFAULT\Software\Sysinternals /v EulaAccepted /t REG_DWORD /d 1 /f}
Start-Process Sysmon64.exe -Argumentlist @("-i", "sysmonconfig.xml")
}
```

Lưu lại file này với tên __run.ps1__

*Lưu ý: phải để tất cả file này vào thư mục SysinternalsSuite đã giải nén thì mới chạy được*

Sau đó chạy file __run.ps1__ với Powershell

Để kiểm tra xem Sysmon đã cài đặt thành công hay chưa, tôi sẽ mở Windows và tìm __Event Viewer__. Sau đó chọn __Applications and Services Logs -> Microsoft -> Windows__. Kéo xuống tìm __Sysmon__, nếu có __Operational__ nghĩa là thành công và có thể kiểm tra log ngay trong đó

### Cài đặt Wazuh-agent cho windows

Trước tiên, trên Wazuh-manager tôi tạo 2 group cho agent của 2 OS là windows và linux. Manager có cơ chế thay đổi được cấu hình ghi nhận log của agent tùy theo cấu hình của group.

Sau khi tạo group với tên __OS-WINDOWS__, tôi sẽ sửa đổi cấu hình của group như bên dưới để có thể nhận log từ __Sysmon__

```python
<agent_config>
	<client_buffer>
		<!-- Agent buffer options -->
		<disabled>no</disabled>
		<queue_size>5000</queue_size>
		<events_per_second>500</events_per_second>
	</client_buffer>
	<!-- Policy monitoring -->
	<rootcheck>
		<disabled>no</disabled>
		<windows_apps>./shared/win_applications_rcl.txt</windows_apps>
		<windows_malware>./shared/win_malware_rcl.txt</windows_malware>
	</rootcheck>
	<sca>
		<enabled>yes</enabled>
		<scan_on_start>yes</scan_on_start>
		<interval>12h</interval>
		<skip_nfs>yes</skip_nfs>
	</sca>
	<!-- File integrity monitoring -->
	<syscheck>
		<disabled>no</disabled>
		<!-- Frequency that syscheck is executed default every 12 hours -->
		<frequency>43200</frequency>
		<!-- Default files to be monitored. -->
		<directories recursion_level="0" restrict="regedit.exe$|system.ini$|win.ini$">%WINDIR%</directories>
		<directories recursion_level="0" restrict="at.exe$|attrib.exe$|cacls.exe$|cmd.exe$|eventcreate.exe$|ftp.exe$|lsass.exe$|net.exe$|net1.exe$|netsh.exe$|reg.exe$|regedt32.exe|regsvr32.exe|runas.exe|sc.exe|schtasks.exe|sethc.exe|subst.exe$">%WINDIR%\SysNative</directories>
		<directories recursion_level="0">%WINDIR%\SysNative\drivers\etc</directories>
		<directories recursion_level="0" restrict="WMIC.exe$">%WINDIR%\SysNative\wbem</directories>
		<directories recursion_level="0" restrict="powershell.exe$">%WINDIR%\SysNative\WindowsPowerShell\v1.0</directories>
		<directories recursion_level="0" restrict="winrm.vbs$">%WINDIR%\SysNative</directories>
		<!-- 32-bit programs. -->
		<directories recursion_level="0" restrict="at.exe$|attrib.exe$|cacls.exe$|cmd.exe$|eventcreate.exe$|ftp.exe$|lsass.exe$|net.exe$|net1.exe$|netsh.exe$|reg.exe$|regedit.exe$|regedt32.exe$|regsvr32.exe$|runas.exe$|sc.exe$|schtasks.exe$|sethc.exe$|subst.exe$">%WINDIR%\System32</directories>
		<directories recursion_level="0">%WINDIR%\System32\drivers\etc</directories>
		<directories recursion_level="0" restrict="WMIC.exe$">%WINDIR%\System32\wbem</directories>
		<directories recursion_level="0" restrict="powershell.exe$">%WINDIR%\System32\WindowsPowerShell\v1.0</directories>
		<directories recursion_level="0" restrict="winrm.vbs$">%WINDIR%\System32</directories>
		<directories realtime="yes">%PROGRAMDATA%\Microsoft\Windows\Start Menu\Programs\Startup</directories>
		<ignore>%PROGRAMDATA%\Microsoft\Windows\Start Menu\Programs\Startup\desktop.ini</ignore>
		<ignore type="sregex">.log$|.htm$|.jpg$|.png$|.chm$|.pnf$|.evtx$</ignore>
		<!-- Windows registry entries to monitor. -->
		<windows_registry>HKEY_LOCAL_MACHINE\Software\Classes\batfile</windows_registry>
		<windows_registry>HKEY_LOCAL_MACHINE\Software\Classes\cmdfile</windows_registry>
		<windows_registry>HKEY_LOCAL_MACHINE\Software\Classes\comfile</windows_registry>
		<windows_registry>HKEY_LOCAL_MACHINE\Software\Classes\exefile</windows_registry>
		<windows_registry>HKEY_LOCAL_MACHINE\Software\Classes\piffile</windows_registry>
		<windows_registry>HKEY_LOCAL_MACHINE\Software\Classes\AllFilesystemObjects</windows_registry>
		<windows_registry>HKEY_LOCAL_MACHINE\Software\Classes\Directory</windows_registry>
		<windows_registry>HKEY_LOCAL_MACHINE\Software\Classes\Folder</windows_registry>
		<windows_registry arch="both">HKEY_LOCAL_MACHINE\Software\Classes\Protocols</windows_registry>
		<windows_registry arch="both">HKEY_LOCAL_MACHINE\Software\Policies</windows_registry>
		<windows_registry>HKEY_LOCAL_MACHINE\Security</windows_registry>
		<windows_registry arch="both">HKEY_LOCAL_MACHINE\Software\Microsoft\Internet Explorer</windows_registry>
		<windows_registry>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services</windows_registry>
		<windows_registry>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Session Manager\KnownDLLs</windows_registry>
		<windows_registry>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\SecurePipeServers\winreg</windows_registry>
		<windows_registry arch="both">HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run</windows_registry>
		<windows_registry arch="both">HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce</windows_registry>
		<windows_registry>HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnceEx</windows_registry>
		<windows_registry arch="both">HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\URL</windows_registry>
		<windows_registry arch="both">HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies</windows_registry>
		<windows_registry arch="both">HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Windows</windows_registry>
		<windows_registry arch="both">HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Winlogon</windows_registry>
		<windows_registry arch="both">HKEY_LOCAL_MACHINE\Software\Microsoft\Active Setup\Installed Components</windows_registry>
		<!-- Windows registry entries to ignore. -->
		<registry_ignore>HKEY_LOCAL_MACHINE\Security\Policy\Secrets</registry_ignore>
		<registry_ignore>HKEY_LOCAL_MACHINE\Security\SAM\Domains\Account\Users</registry_ignore>
		<registry_ignore type="sregex">\Enum$</registry_ignore>
		<registry_ignore>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\MpsSvc\Parameters\AppCs</registry_ignore>
		<registry_ignore>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\MpsSvc\Parameters\PortKeywords\DHCP</registry_ignore>
		<registry_ignore>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\MpsSvc\Parameters\PortKeywords\IPTLSIn</registry_ignore>
		<registry_ignore>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\MpsSvc\Parameters\PortKeywords\IPTLSOut</registry_ignore>
		<registry_ignore>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\MpsSvc\Parameters\PortKeywords\RPC-EPMap</registry_ignore>
		<registry_ignore>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\MpsSvc\Parameters\PortKeywords\Teredo</registry_ignore>
		<registry_ignore>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\PolicyAgent\Parameters\Cache</registry_ignore>
		<registry_ignore>HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnceEx</registry_ignore>
		<registry_ignore>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\ADOVMPPackage\Final</registry_ignore>
		<!-- Frequency for ACL checking (seconds) -->
		<windows_audit_interval>60</windows_audit_interval>
		<!-- Nice value for Syscheck module -->
		<process_priority>10</process_priority>
		<!-- Maximum output throughput -->
		<max_eps>100</max_eps>
		<!-- Database synchronization settings -->
		<synchronization>
			<enabled>yes</enabled>
			<interval>5m</interval>
			<max_interval>1h</max_interval>
			<max_eps>10</max_eps>
		</synchronization>
	</syscheck>
	<!-- System inventory -->
	<wodle name="syscollector">
		<disabled>no</disabled>
		<interval>1h</interval>
		<scan_on_start>yes</scan_on_start>
		<hardware>yes</hardware>
		<os>yes</os>
		<network>yes</network>
		<packages>yes</packages>
		<ports all="no">yes</ports>
		<processes>yes</processes>
		<!-- Database synchronization settings -->
		<synchronization>
			<max_eps>10</max_eps>
		</synchronization>
	</wodle>
	<!-- CIS policies evaluation -->
	<wodle name="cis-cat">
		<disabled>no</disabled>
		<timeout>1800</timeout>
		<interval>1d</interval>
		<scan-on-start>yes</scan-on-start>
		<java_path>\\server\jre\bin\java.exe</java_path>
		<ciscat_path>C:\cis-cat</ciscat_path>
	</wodle>
	<!-- Osquery integration -->
	<wodle name="osquery">
		<disabled>yes</disabled>
		<run_daemon>yes</run_daemon>
		<bin_path>C:\Program Files\osquery\osqueryd</bin_path>
		<log_path>C:\Program Files\osquery\log\osqueryd.results.log</log_path>
		<config_path>C:\Program Files\osquery\osquery.conf</config_path>
		<add_labels>yes</add_labels>
	</wodle>
	<!-- Active response -->
	<active-response>
		<disabled>no</disabled>
		<ca_store>wpk_root.pem</ca_store>
		<ca_verification>yes</ca_verification>
	</active-response>
	<!-- Log analysis -->
	<localfile>
		<location>Microsoft-Windows-Sysmon/Operational</location>
		<log_format>eventchannel</log_format>
	</localfile>
	<localfile>
		<location>Windows PowerShell</location>
		<log_format>eventchannel</log_format>
	</localfile>
	<localfile>
		<location>Microsoft-Windows-CodeIntegrity/Operational</location>
		<log_format>eventchannel</log_format>
	</localfile>
	<localfile>
		<location>Microsoft-Windows-TaskScheduler/Operational</location>
		<log_format>eventchannel</log_format>
	</localfile>
	<localfile>
		<location>Microsoft-Windows-PowerShell/Operational</location>
		<log_format>eventchannel</log_format>
	</localfile>
	<localfile>
		<location>Microsoft-Windows-Windows Firewall With Advanced Security/Firewall</location>
		<log_format>eventchannel</log_format>
	</localfile>
	<localfile>
		<location>Microsoft-Windows-Windows Defender/Operational</location>
		<log_format>eventchannel</log_format>
	</localfile>
</agent_config>
```

Kiểm tra lại file cấu hình ở trên, nếu có phần 

```python
	<localfile>
		<location>Microsoft-Windows-Sysmon/Operational</location>
		<log_format>eventchannel</log_format>
	</localfile>
```

thì manager mới nhận được log từ __Sysmon__

Tiếp theo, tôi tải Wazuh-agent cho Windows [từ trang chủ của Wazuh](https://documentation.wazuh.com/current/installation-guide/packages-list.html) và chạy nó với quyền Admin.

Sau khi cài đặt xong, có 1 thư mục tên __ossec-agent__ sẽ xuất hiện trong __Program Files (x86)__. File cấu hình của agent sẽ nằm trong thư mục này có tên __ossec.conf__.

Tôi sẽ sửa đổi nó 1 chút để có thể nhận IP của Wazuh-manager và chui vào group __OS-WINDOWS__ mà tôi vừa tạo phía trên. Chỉ cần thay đổi _address_ cho đúng với IP của manager.

```python
<!--
  Wazuh - Agent - Default configuration for Windows
  More info at: https://documentation.wazuh.com
  Mailing list: https://groups.google.com/forum/#!forum/wazuh
-->

<ossec_config>

  <client>
    <server>
      <address>10.10.24.1</address>       
      <port>1514</port>
      <protocol>tcp</protocol>
    </server>
    <config-profile>windows, windows10</config-profile>
    <crypto_method>aes</crypto_method>
    <notify_time>10</notify_time>
    <time-reconnect>60</time-reconnect>
    <auto_restart>yes</auto_restart>
    <enrollment>
        <enabled>yes</enabled>
        <manager_address>10.10.24.1</manager_address>
        <groups>OS-WINDOWS</groups>
    </enrollment>
  </client>


  <!-- Agent buffer options -->
  <client_buffer>
    <disabled>no</disabled>
    <queue_size>5000</queue_size>
    <events_per_second>500</events_per_second>
  </client_buffer>

  <!-- Log analysis -->
  <localfile>
    <location>Application</location>
    <log_format>eventchannel</log_format>
  </localfile>

  <localfile>
    <location>Security</location>
    <log_format>eventchannel</log_format>
    <query>Event/System[EventID != 5145 and EventID != 5156 and EventID != 5447 and
      EventID != 4656 and EventID != 4658 and EventID != 4663 and EventID != 4660 and
      EventID != 4670 and EventID != 4690 and EventID != 4703 and EventID != 4907 and
      EventID != 5152 and EventID != 5157]</query>
  </localfile>

  <localfile>
    <location>System</location>
    <log_format>eventchannel</log_format>
  </localfile>

  <localfile>
    <location>active-response\active-responses.log</location>
    <log_format>syslog</log_format>
  </localfile>

  <!-- Policy monitoring -->
  <rootcheck>
    <disabled>no</disabled>
    <windows_apps>./shared/win_applications_rcl.txt</windows_apps>
    <windows_malware>./shared/win_malware_rcl.txt</windows_malware>
  </rootcheck>

  <!-- Security Configuration Assessment -->
  <sca>
    <enabled>yes</enabled>
    <scan_on_start>yes</scan_on_start>
    <interval>12h</interval>
    <skip_nfs>yes</skip_nfs>
  </sca>

  <!-- File integrity monitoring -->
  <syscheck>

    <disabled>no</disabled>

    <!-- Frequency that syscheck is executed default every 12 hours -->
    <frequency>43200</frequency>

    <!-- Default files to be monitored. -->
    <directories recursion_level="0" restrict="regedit.exe$|system.ini$|win.ini$">%WINDIR%</directories>

    <directories recursion_level="0" restrict="at.exe$|attrib.exe$|cacls.exe$|cmd.exe$|eventcreate.exe$|ftp.exe$|lsass.exe$|net.exe$|net1.exe$|netsh.exe$|reg.exe$|regedt32.exe|regsvr32.exe|runas.exe|sc.exe|schtasks.exe|sethc.exe|subst.exe$">%WINDIR%\SysNative</directories>
    <directories recursion_level="0">%WINDIR%\SysNative\drivers\etc</directories>
    <directories recursion_level="0" restrict="WMIC.exe$">%WINDIR%\SysNative\wbem</directories>
    <directories recursion_level="0" restrict="powershell.exe$">%WINDIR%\SysNative\WindowsPowerShell\v1.0</directories>
    <directories recursion_level="0" restrict="winrm.vbs$">%WINDIR%\SysNative</directories>

    <!-- 32-bit programs. -->
    <directories recursion_level="0" restrict="at.exe$|attrib.exe$|cacls.exe$|cmd.exe$|eventcreate.exe$|ftp.exe$|lsass.exe$|net.exe$|net1.exe$|netsh.exe$|reg.exe$|regedit.exe$|regedt32.exe$|regsvr32.exe$|runas.exe$|sc.exe$|schtasks.exe$|sethc.exe$|subst.exe$">%WINDIR%\System32</directories>
    <directories recursion_level="0">%WINDIR%\System32\drivers\etc</directories>
    <directories recursion_level="0" restrict="WMIC.exe$">%WINDIR%\System32\wbem</directories>
    <directories recursion_level="0" restrict="powershell.exe$">%WINDIR%\System32\WindowsPowerShell\v1.0</directories>
    <directories recursion_level="0" restrict="winrm.vbs$">%WINDIR%\System32</directories>

    <directories realtime="yes">%PROGRAMDATA%\Microsoft\Windows\Start Menu\Programs\Startup</directories>

    <ignore>%PROGRAMDATA%\Microsoft\Windows\Start Menu\Programs\Startup\desktop.ini</ignore>

    <ignore type="sregex">.log$|.htm$|.jpg$|.png$|.chm$|.pnf$|.evtx$</ignore>

    <!-- Windows registry entries to monitor. -->
    <windows_registry>HKEY_LOCAL_MACHINE\Software\Classes\batfile</windows_registry>
    <windows_registry>HKEY_LOCAL_MACHINE\Software\Classes\cmdfile</windows_registry>
    <windows_registry>HKEY_LOCAL_MACHINE\Software\Classes\comfile</windows_registry>
    <windows_registry>HKEY_LOCAL_MACHINE\Software\Classes\exefile</windows_registry>
    <windows_registry>HKEY_LOCAL_MACHINE\Software\Classes\piffile</windows_registry>
    <windows_registry>HKEY_LOCAL_MACHINE\Software\Classes\AllFilesystemObjects</windows_registry>
    <windows_registry>HKEY_LOCAL_MACHINE\Software\Classes\Directory</windows_registry>
    <windows_registry>HKEY_LOCAL_MACHINE\Software\Classes\Folder</windows_registry>
    <windows_registry arch="both">HKEY_LOCAL_MACHINE\Software\Classes\Protocols</windows_registry>
    <windows_registry arch="both">HKEY_LOCAL_MACHINE\Software\Policies</windows_registry>
    <windows_registry>HKEY_LOCAL_MACHINE\Security</windows_registry>
    <windows_registry arch="both">HKEY_LOCAL_MACHINE\Software\Microsoft\Internet Explorer</windows_registry>

    <windows_registry>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services</windows_registry>
    <windows_registry>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Session Manager\KnownDLLs</windows_registry>
    <windows_registry>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\SecurePipeServers\winreg</windows_registry>

    <windows_registry arch="both">HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run</windows_registry>
    <windows_registry arch="both">HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce</windows_registry>
    <windows_registry>HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnceEx</windows_registry>
    <windows_registry arch="both">HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\URL</windows_registry>
    <windows_registry arch="both">HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies</windows_registry>
    <windows_registry arch="both">HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Windows</windows_registry>
    <windows_registry arch="both">HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Winlogon</windows_registry>

    <windows_registry arch="both">HKEY_LOCAL_MACHINE\Software\Microsoft\Active Setup\Installed Components</windows_registry>

    <!-- Windows registry entries to ignore. -->
    <registry_ignore>HKEY_LOCAL_MACHINE\Security\Policy\Secrets</registry_ignore>
    <registry_ignore>HKEY_LOCAL_MACHINE\Security\SAM\Domains\Account\Users</registry_ignore>
    <registry_ignore type="sregex">\Enum$</registry_ignore>
    <registry_ignore>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\MpsSvc\Parameters\AppCs</registry_ignore>
    <registry_ignore>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\MpsSvc\Parameters\PortKeywords\DHCP</registry_ignore>
    <registry_ignore>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\MpsSvc\Parameters\PortKeywords\IPTLSIn</registry_ignore>
    <registry_ignore>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\MpsSvc\Parameters\PortKeywords\IPTLSOut</registry_ignore>
    <registry_ignore>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\MpsSvc\Parameters\PortKeywords\RPC-EPMap</registry_ignore>
    <registry_ignore>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\MpsSvc\Parameters\PortKeywords\Teredo</registry_ignore>
    <registry_ignore>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\PolicyAgent\Parameters\Cache</registry_ignore>
    <registry_ignore>HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnceEx</registry_ignore>
    <registry_ignore>HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\ADOVMPPackage\Final</registry_ignore>

    <!-- Frequency for ACL checking (seconds) -->
    <windows_audit_interval>60</windows_audit_interval>

    <!-- Nice value for Syscheck module -->
    <process_priority>10</process_priority>

    <!-- Maximum output throughput -->
    <max_eps>100</max_eps>

    <!-- Database synchronization settings -->
    <synchronization>
      <enabled>yes</enabled>
      <interval>5m</interval>
      <max_interval>1h</max_interval>
      <max_eps>10</max_eps>
    </synchronization>
  </syscheck>

  <!-- System inventory -->
  <wodle name="syscollector">
    <disabled>no</disabled>
    <interval>1h</interval>
    <scan_on_start>yes</scan_on_start>
    <hardware>yes</hardware>
    <os>yes</os>
    <network>yes</network>
    <packages>yes</packages>
    <ports all="no">yes</ports>
    <processes>yes</processes>

    <!-- Database synchronization settings -->
    <synchronization>
      <max_eps>10</max_eps>
    </synchronization>
  </wodle>

  <!-- CIS policies evaluation -->
  <wodle name="cis-cat">
    <disabled>yes</disabled>
    <timeout>1800</timeout>
    <interval>1d</interval>
    <scan-on-start>yes</scan-on-start>

    <java_path>\\server\jre\bin\java.exe</java_path>
    <ciscat_path>C:\cis-cat</ciscat_path>
  </wodle>

  <!-- Osquery integration -->
  <wodle name="osquery">
    <disabled>yes</disabled>
    <run_daemon>yes</run_daemon>
    <bin_path>C:\Program Files\osquery\osqueryd</bin_path>
    <log_path>C:\Program Files\osquery\log\osqueryd.results.log</log_path>
    <config_path>C:\Program Files\osquery\osquery.conf</config_path>
    <add_labels>yes</add_labels>
  </wodle>

  <!-- Active response -->
  <active-response>
    <disabled>no</disabled>
    <ca_store>wpk_root.pem</ca_store>
    <ca_verification>yes</ca_verification>
  </active-response>

  <!-- Choose between plain or json format (or both) for internal logs -->
  <logging>
    <log_format>plain</log_format>
  </logging>

</ossec_config>

<!-- END of Default Configuration. -->
```

Lưu file lại. Sau đó chạy __win32ui.exe__ với quyền admin. Chọn __Manage -> Restart__. Chờ 1 lúc để hệ thống tạo kết nối và vào Manager để kiểm tra lại.

## Linux OS

### Packetbeat

[Packetbeat](https://www.elastic.co/guide/en/beats/packetbeat/current/packetbeat-overview.html) là một công cụ phân tích lưu lượng mạng và hiệu năng hệ thống. Nó hoạt động bằng cách chụp lại các traffic mạng giữa các ứng dụng, dịch vụ, và tất nhiên là cả những traffic ra và vào internet. 

Ví dụ tôi đang có máy Ubuntu 22.04 và muốn cài agent lên đây thì tôi sẽ [cài Packetbeat bằng apt](https://www.elastic.co/guide/en/beats/packetbeat/8.8/setup-repositories.html#_apt). 

Tuy nhiên tôi sẽ thay đổi cấu hình của __Packetbeat__ một chút theo mục đích của tôi. Tôi sẽ thay đổi output của __Packetbeat__ để manager có thể lấy log của nó dễ dàng hơn

```python
#################### Packetbeat Configuration Example #########################

# This file is an example configuration file highlighting only the most common
# options. The packetbeat.reference.yml file from the same directory contains all the
# supported options with more comments. You can use it as a reference.
#
# You can find the full configuration reference here:
# https://www.elastic.co/guide/en/beats/packetbeat/index.html

# =============================== Network device ===============================

# Select the network interface to sniff the data. On Linux, you can use the
# "any" keyword to sniff on all connected interfaces.
packetbeat.interfaces.device: any

# The network CIDR blocks that are considered "internal" networks for
# the purpose of network perimeter boundary classification. The valid
# values for internal_networks are the same as those that can be used
# with processor network conditions.
#
# For a list of available values see:
# https://www.elastic.co/guide/en/beats/packetbeat/current/defining-processors.html#condition-network
packetbeat.interfaces.internal_networks:
  - private

# =================================== Flows ====================================

# Set `enabled: false` or comment out all options to disable flows reporting.
packetbeat.flows:
  # Set network flow timeout. Flow is killed if no packet is received before being
  # timed out.
  timeout: 30s

  # Configure reporting period. If set to -1, only killed flows will be reported
  period: 10s

# =========================== Transaction protocols ============================

packetbeat.protocols:
- type: icmp
  # Enable ICMPv4 and ICMPv6 monitoring. The default is true.
  enabled: true

- type: amqp
  # Configure the ports where to listen for AMQP traffic. You can disable
  # the AMQP protocol by commenting out the list of ports.
  ports: [5672]

- type: cassandra
  # Configure the ports where to listen for Cassandra traffic. You can disable
  # the Cassandra protocol by commenting out the list of ports.
  ports: [9042]

- type: dhcpv4
  # Configure the DHCP for IPv4 ports.
  ports: [67, 68]

- type: dns
  # Configure the ports where to listen for DNS traffic. You can disable
  # the DNS protocol by commenting out the list of ports.
  ports: [53]

- type: http
  # Configure the ports where to listen for HTTP traffic. You can disable
  # the HTTP protocol by commenting out the list of ports.
  ports: [80, 8080, 8000, 5000, 8002]

- type: memcache
  # Configure the ports where to listen for memcache traffic. You can disable
  # the Memcache protocol by commenting out the list of ports.
  ports: [11211]

- type: mysql
  # Configure the ports where to listen for MySQL traffic. You can disable
  # the MySQL protocol by commenting out the list of ports.
  ports: [3306,3307]

- type: pgsql
  # Configure the ports where to listen for Pgsql traffic. You can disable
  # the Pgsql protocol by commenting out the list of ports.
  ports: [5432]

- type: redis
  # Configure the ports where to listen for Redis traffic. You can disable
  # the Redis protocol by commenting out the list of ports.
  ports: [6379]

- type: thrift
  # Configure the ports where to listen for Thrift-RPC traffic. You can disable
  # the Thrift-RPC protocol by commenting out the list of ports.
  ports: [9090]

- type: mongodb
  # Configure the ports where to listen for MongoDB traffic. You can disable
  # the MongoDB protocol by commenting out the list of ports.
  ports: [27017]

- type: nfs
  # Configure the ports where to listen for NFS traffic. You can disable
  # the NFS protocol by commenting out the list of ports.
  ports: [2049]

- type: tls
  # Configure the ports where to listen for TLS traffic. You can disable
  # the TLS protocol by commenting out the list of ports.
  ports:
    - 443   # HTTPS
    - 993   # IMAPS
    - 995   # POP3S
    - 5223  # XMPP over SSL
    - 8443
    - 8883  # Secure MQTT
    - 9243  # Elasticsearch

- type: sip
  # Configure the ports where to listen for SIP traffic. You can disable
  # the SIP protocol by commenting out the list of ports.
  ports: [5060]

# ======================= Elasticsearch template setting =======================

setup.template.settings:
  index.number_of_shards: 1
  #index.codec: best_compression
  #_source.enabled: false

# ================================== General ===================================

# The name of the shipper that publishes the network data. It can be used to group
# all the transactions sent by a single shipper in the web interface.
#name:

# A list of tags to include in every event. In the default configuration file
# the forwarded tag causes Packetbeat to not add any host fields. If you are
# monitoring a network tap or mirror port then add the forwarded tag.
#tags: [forwarded]

# Optional fields that you can specify to add additional information to the
# output.
#fields:
#  env: staging

# ================================= Dashboards =================================
# These settings control loading the sample dashboards to the Kibana index. Loading
# the dashboards is disabled by default and can be enabled either by setting the
# options here or by using the `setup` command.
#setup.dashboards.enabled: false

# The URL from where to download the dashboards archive. By default this URL
# has a value which is computed based on the Beat name and version. For released
# versions, this URL points to the dashboard archive on the artifacts.elastic.co
# website.
#setup.dashboards.url:

# =================================== Kibana ===================================

# Starting with Beats version 6.0.0, the dashboards are loaded via the Kibana API.
# This requires a Kibana endpoint configuration.
setup.kibana:

  # Kibana Host
  # Scheme and port can be left out and will be set to the default (http and 5601)
  # In case you specify and additional path, the scheme is required: http://localhost:5601/path
  # IPv6 addresses should always be defined as: https://[2001:db8::1]:5601
  #host: "localhost:5601"

  # Kibana Space ID
  # ID of the Kibana Space into which the dashboards should be loaded. By default,
  # the Default Space will be used.
  #space.id:

# =============================== Elastic Cloud ================================

# These settings simplify using Packetbeat with the Elastic Cloud (https://cloud.elastic.co/).

# The cloud.id setting overwrites the `output.elasticsearch.hosts` and
# `setup.kibana.host` options.
# You can find the `cloud.id` in the Elastic Cloud web UI.
#cloud.id:

# The cloud.auth setting overwrites the `output.elasticsearch.username` and
# `output.elasticsearch.password` settings. The format is `<user>:<pass>`.
#cloud.auth:

# ================================== Outputs ===================================

# Configure what output to use when sending the data collected by the beat.

# ---------------------------- Elasticsearch Output ----------------------------
#output.elasticsearch:
  # Array of hosts to connect to.
#  hosts: ["localhost:9200"]

  # Protocol - either `http` (default) or `https`.
  #protocol: "https"

  # Authentication credentials - either API key or username/password.
  #api_key: "id:api_key"
  #username: "elastic"
  #password: "changeme"
  
output.file:
 path: "/tmp/packetbeat"
 filename: packetbeat
# ------------------------------ Logstash Output -------------------------------
#output.logstash:
  # The Logstash hosts
  #hosts: ["localhost:5044"]

  # Optional SSL. By default is off.
  # List of root certificates for HTTPS server verifications
  #ssl.certificate_authorities: ["/etc/pki/root/ca.pem"]

  # Certificate for SSL client authentication
  #ssl.certificate: "/etc/pki/client/cert.pem"

  # Client Certificate Key
  #ssl.key: "/etc/pki/client/cert.key"

# ================================= Processors =================================

processors:
  - # Add forwarded to tags when processing data from a network tap or mirror.
    if.contains.tags: forwarded
    then:
      - drop_fields:
          fields: [host]
    else:
      - add_host_metadata: ~
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
  - detect_mime_type:
      field: http.request.body.content
      target: http.request.mime_type
  - detect_mime_type:
      field: http.response.body.content
      target: http.response.mime_type

# ================================== Logging ===================================

# Sets log level. The default log level is info.
# Available log levels are: error, warning, info, debug
#logging.level: debug

# At debug level, you can selectively enable logging only for some components.
# To enable all selectors use ["*"]. Examples of other selectors are "beat",
# "publisher", "service".
#logging.selectors: ["*"]

# ============================= X-Pack Monitoring ==============================
# Packetbeat can export internal metrics to a central Elasticsearch monitoring
# cluster.  This requires xpack monitoring to be enabled in Elasticsearch.  The
# reporting is disabled by default.

# Set to true to enable the monitoring reporter.
#monitoring.enabled: false

# Sets the UUID of the Elasticsearch cluster under which monitoring data for this
# Packetbeat instance will appear in the Stack Monitoring UI. If output.elasticsearch
# is enabled, the UUID is derived from the Elasticsearch cluster referenced by output.elasticsearch.
#monitoring.cluster_uuid:

# Uncomment to send the metrics to Elasticsearch. Most settings from the
# Elasticsearch output are accepted here as well.
# Note that the settings should point to your Elasticsearch *monitoring* cluster.
# Any setting that is not set is automatically inherited from the Elasticsearch
# output configuration, so if you have the Elasticsearch output configured such
# that it is pointing to your Elasticsearch monitoring cluster, you can simply
# uncomment the following line.
#monitoring.elasticsearch:

# ============================== Instrumentation ===============================

# Instrumentation support for the packetbeat.
#instrumentation:
    # Set to true to enable instrumentation of packetbeat.
    #enabled: false

    # Environment in which packetbeat is running on (eg: staging, production, etc.)
    #environment: ""

    # APM Server hosts to report instrumentation results to.
    #hosts:
    #  - http://localhost:8200

    # API Key for the APM Server(s).
    # If api_key is set then secret_token will be ignored.
    #api_key:

    # Secret token for the APM Server(s).
    #secret_token:


# ================================= Migration ==================================

# This allows to enable 6.7 migration aliases
#migration.6_to_7.enabled: true
```

Lưu lại file và khởi động lại dịch vụ

```python
systemctl restart packetbeat
```

### Cài đặt Wazuh-agent cho linux

Như đã tạo group __OS-LINUX__ cho manager ở trên nên bây giờ tôi chỉ cần thay đổi cấu hình của group này

```python
<agent_config>
	<client_buffer>
		<!-- Agent buffer options -->
		<disabled>no</disabled>
		<queue_size>5000</queue_size>
		<events_per_second>500</events_per_second>
	</client_buffer>
	<!-- Policy monitoring -->
	<rootcheck>
		<disabled>no</disabled>
		<!-- Frequency that rootcheck is executed - every 12 hours -->
		<frequency>43200</frequency>
		<rootkit_files>/var/ossec/etc/shared/rootkit_files.txt</rootkit_files>
		<rootkit_trojans>/var/ossec/etc/shared/rootkit_trojans.txt</rootkit_trojans>
		<system_audit>/var/ossec/etc/shared/system_audit_rcl.txt</system_audit>
		<system_audit>/var/ossec/etc/shared/system_audit_ssh.txt</system_audit>
		<system_audit>/var/ossec/etc/shared/cis_debian_linux_rcl.txt</system_audit>
		<skip_nfs>yes</skip_nfs>
	</rootcheck>
	<wodle name="open-scap">
		<disabled>yes</disabled>
		<timeout>1800</timeout>
		<interval>1d</interval>
		<scan-on-start>yes</scan-on-start>
		<content type="xccdf" path="ssg-debian-8-ds.xml">
			<profile>xccdf_org.ssgproject.content_profile_common</profile>
		</content>
		<content type="oval" path="cve-debian-oval.xml"/>
	</wodle>
	<!-- File integrity monitoring -->
	<syscheck>
		<disabled>no</disabled>
		<!-- Frequency that syscheck is executed default every 12 hours -->
		<frequency>43200</frequency>
		<scan_on_start>yes</scan_on_start>
		<!-- Directories to check  (perform all possible verifications) -->
		<directories>/etc,/usr/bin,/usr/sbin</directories>
		<directories>/bin,/sbin,/boot</directories>
		<!-- Files/directories to ignore -->
		<ignore>/etc/mtab</ignore>
		<ignore>/etc/hosts.deny</ignore>
		<ignore>/etc/mail/statistics</ignore>
		<ignore>/etc/random-seed</ignore>
		<ignore>/etc/random.seed</ignore>
		<ignore>/etc/adjtime</ignore>
		<ignore>/etc/httpd/logs</ignore>
		<ignore>/etc/utmpx</ignore>
		<ignore>/etc/wtmpx</ignore>
		<ignore>/etc/cups/certs</ignore>
		<ignore>/etc/dumpdates</ignore>
		<ignore>/etc/svc/volatile</ignore>
		<ignore>/sys/kernel/security</ignore>
		<ignore>/sys/kernel/debug</ignore>
		<!-- File types to ignore -->
		<ignore type="sregex">.log$|.swp$</ignore>
		<!-- Check the file, but never compute the diff -->
		<nodiff>/etc/ssl/private.key</nodiff>
		<skip_nfs>yes</skip_nfs>
		<skip_dev>yes</skip_dev>
		<skip_proc>yes</skip_proc>
		<skip_sys>yes</skip_sys>
		<!-- Nice value for Syscheck process -->
		<process_priority>10</process_priority>
		<!-- Maximum output throughput -->
		<max_eps>100</max_eps>
		<!-- Database synchronization settings -->
		<synchronization>
			<enabled>yes</enabled>
			<interval>5m</interval>
			<response_timeout>30</response_timeout>
			<queue_size>16384</queue_size>
			<max_eps>10</max_eps>
		</synchronization>
	</syscheck>
	<!-- Log analysis -->
	<localfile>
		<log_format>json</log_format>
		<location>/tmp/packetbeat/packetbeat*</location>
	</localfile>
	<localfile>
		<log_format>syslog</log_format>
		<location>/var/ossec/logs/active-responses.log</location>
	</localfile>
	<localfile>
		<log_format>syslog</log_format>
		<location>/var/log/messages</location>
	</localfile>
	<localfile>
		<log_format>syslog</log_format>
		<location>/var/log/auth.log</location>
	</localfile>
	<localfile>
		<log_format>syslog</log_format>
		<location>/var/log/syslog</location>
	</localfile>
	<localfile>
		<log_format>command</log_format>
		<command>df -P</command>
		<frequency>360</frequency>
	</localfile>
	<localfile>
		<log_format>full_command</log_format>
		<command>netstat -tan |grep LISTEN |grep -v 127.0.0.1 | sort</command>
		<frequency>360</frequency>
	</localfile>
	<localfile>
		<log_format>full_command</log_format>
		<command>last -n 5</command>
		<frequency>360</frequency>
	</localfile>
	<wodle name="osquery">
		<disabled>yes</disabled>
		<run_daemon>yes</run_daemon>
		<log_path>/var/log/osquery/osqueryd.results.log</log_path>
		<config_path>/etc/osquery/osquery.conf</config_path>
		<add_labels>yes</add_labels>
	</wodle>
	<wodle name="syscollector">
		<disabled>no</disabled>
		<interval>24h</interval>
		<scan_on_start>yes</scan_on_start>
		<packages>yes</packages>
		<os>yes</os>
		<hotfixes>yes</hotfixes>
		<ports all="no">yes</ports>
		<processes>yes</processes>
	</wodle>
</agent_config>
```

Kiểm tra lại file cấu hình ở trên, nếu có phần 

```python
	<localfile>
		<log_format>json</log_format>
		<location>/tmp/packetbeat/packetbeat*</location>
	</localfile>
```

thì manager mới nhận được log từ __Packetbeat__

Tiếp theo tôi sẽ cài đặt Wazuh-agent cho Linux theo [hướng dẫn của Wazuh](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-linux.html). Tuy nhiên tôi sẽ sửa cấu hình của agent theo mục đích của mình.

Đầu tiên tôi xóa file config mặc định

```python
rm -rf /var/ossec/etc/ossec.conf
```

Tạo file config mới và thêm cấu hình. Thay phần _address_ bằng IP của manager

```python
nano /var/ossec/etc/ossec.conf
```

```python
<!--
  Wazuh - Agent - Default configuration for rhel 7.9
  More info at: https://documentation.wazuh.com
  Mailing list: https://groups.google.com/forum/#!forum/wazuh
-->

<ossec_config>
  <client>
    <server>
      <address>10.10.24.1</address>
      <port>1514</port>
      <protocol>tcp</protocol>
    </server>
    <config-profile>linux</config-profile>
    <notify_time>10</notify_time>
    <time-reconnect>60</time-reconnect>
    <auto_restart>yes</auto_restart>
    <crypto_method>aes</crypto_method>
    <notify_time>10</notify_time>
    <time-reconnect>60</time-reconnect>
    <auto_restart>yes</auto_restart>
    <enrollment>
        <enabled>yes</enabled>
        <manager_address>10.10.24.1</manager_address>
        <groups>OS-LINUX</groups>
    </enrollment> 
  </client>

  <client_buffer>
    <!-- Agent buffer options -->
    <disabled>no</disabled>
    <queue_size>5000</queue_size>
    <events_per_second>500</events_per_second>
  </client_buffer>

  <!-- Policy monitoring -->
  <rootcheck>
    <disabled>no</disabled>
    <check_files>yes</check_files>
    <check_trojans>yes</check_trojans>
    <check_dev>yes</check_dev>
    <check_sys>yes</check_sys>
    <check_pids>yes</check_pids>
    <check_ports>yes</check_ports>
    <check_if>yes</check_if>

    <!-- Frequency that rootcheck is executed - every 12 hours -->
    <frequency>43200</frequency>

    <rootkit_files>etc/shared/rootkit_files.txt</rootkit_files>
    <rootkit_trojans>etc/shared/rootkit_trojans.txt</rootkit_trojans>

    <skip_nfs>yes</skip_nfs>
  </rootcheck>

  <wodle name="cis-cat">
    <disabled>yes</disabled>
    <timeout>1800</timeout>
    <interval>1d</interval>
    <scan-on-start>yes</scan-on-start>

    <java_path>wodles/java</java_path>
    <ciscat_path>wodles/ciscat</ciscat_path>
  </wodle>

  <!-- Osquery integration -->
  <wodle name="osquery">
    <disabled>yes</disabled>
    <run_daemon>yes</run_daemon>
    <log_path>/var/log/osquery/osqueryd.results.log</log_path>
    <config_path>/etc/osquery/osquery.conf</config_path>
    <add_labels>yes</add_labels>
  </wodle>

  <!-- System inventory -->
  <wodle name="syscollector">
    <disabled>no</disabled>
    <interval>1h</interval>
    <scan_on_start>yes</scan_on_start>
    <hardware>yes</hardware>
    <os>yes</os>
    <network>yes</network>
    <packages>yes</packages>
    <ports all="no">yes</ports>
    <processes>yes</processes>

    <!-- Database synchronization settings -->
    <synchronization>
      <max_eps>10</max_eps>
    </synchronization>
  </wodle>

  <sca>
    <enabled>yes</enabled>
    <scan_on_start>yes</scan_on_start>
    <interval>12h</interval>
    <skip_nfs>yes</skip_nfs>
  </sca>

  <!-- File integrity monitoring -->
  <syscheck>
    <disabled>no</disabled>

    <!-- Frequency that syscheck is executed default every 12 hours -->
    <frequency>43200</frequency>

    <scan_on_start>yes</scan_on_start>

    <!-- Directories to check  (perform all possible verifications) -->
    <directories>/etc,/usr/bin,/usr/sbin</directories>
    <directories>/bin,/sbin,/boot</directories>

    <!-- Files/directories to ignore -->
    <ignore>/etc/mtab</ignore>
    <ignore>/etc/hosts.deny</ignore>
    <ignore>/etc/mail/statistics</ignore>
    <ignore>/etc/random-seed</ignore>
    <ignore>/etc/random.seed</ignore>
    <ignore>/etc/adjtime</ignore>
    <ignore>/etc/httpd/logs</ignore>
    <ignore>/etc/utmpx</ignore>
    <ignore>/etc/wtmpx</ignore>
    <ignore>/etc/cups/certs</ignore>
    <ignore>/etc/dumpdates</ignore>
    <ignore>/etc/svc/volatile</ignore>

    <!-- File types to ignore -->
    <ignore type="sregex">.log$|.swp$</ignore>

    <!-- Check the file, but never compute the diff -->
    <nodiff>/etc/ssl/private.key</nodiff>

    <skip_nfs>yes</skip_nfs>
    <skip_dev>yes</skip_dev>
    <skip_proc>yes</skip_proc>
    <skip_sys>yes</skip_sys>

    <!-- Nice value for Syscheck process -->
    <process_priority>10</process_priority>

    <!-- Maximum output throughput -->
    <max_eps>100</max_eps>

    <!-- Database synchronization settings -->
    <synchronization>
      <enabled>yes</enabled>
      <interval>5m</interval>
      <max_interval>1h</max_interval>
      <max_eps>10</max_eps>
    </synchronization>
  </syscheck>

  <!-- Log analysis -->
  <localfile>
    <log_format>command</log_format>
    <command>df -P</command>
    <frequency>360</frequency>
  </localfile>

  <localfile>
    <log_format>full_command</log_format>
    <command>netstat -tulpn | sed 's/\([[:alnum:]]\+\)\ \+[[:digit:]]\+\ \+[[:digit:]]\+\ \+\(.*\):\([[:digit:]]*\)\ \+\([0-9\.\:\*]\+\).\+\ \([[:digit:]]*\/[[:alnum:]\-]*\).*/\1 \2 == \3 == \4 \5/' | sort -k 4 -g | sed 's/ == \(.*\) ==/:\1/' | sed 1,2d</command>
    <alias>netstat listening ports</alias>
    <frequency>360</frequency>
  </localfile>

  <localfile>
    <log_format>full_command</log_format>
    <command>last -n 20</command>
    <frequency>360</frequency>
  </localfile>

  <!-- Active response -->
  <active-response>
    <disabled>no</disabled>
    <ca_store>etc/wpk_root.pem</ca_store>
    <ca_verification>yes</ca_verification>
  </active-response>

  <!-- Choose between "plain", "json", or "plain,json" for the format of internal logs -->
  <logging>
    <log_format>plain</log_format>
  </logging>

</ossec_config>

<ossec_config>
  <localfile>
    <log_format>apache</log_format>
    <location>/var/log/httpd/error_log</location>
  </localfile>

  <localfile>
    <log_format>apache</log_format>
    <location>/var/log/httpd/access_log</location>
  </localfile>

  <localfile>
    <log_format>audit</log_format>
    <location>/var/log/audit/audit.log</location>
  </localfile>

  <localfile>
    <log_format>syslog</log_format>
    <location>/var/ossec/logs/active-responses.log</location>
  </localfile>

  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/messages</location>
  </localfile>

  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/secure</location>
  </localfile>

  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/maillog</location>
  </localfile>

  <localfile>
	<log_format>json</log_format>
	<location>/tmp/packetbeat/packetbeat</location>
  </localfile> 
</ossec_config>

<ossec_config>
  <localfile>
    <log_format>full_command</log_format>
    <alias>process list</alias>
    <command>ps -e -o pid,uname,command</command>
    <frequency>30</frequency>
  </localfile>
</ossec_config>
```

Lưu lại file và khởi động lại dịch vụ

```python
systemctl restart wazuh-agent
```

Chờ 1 lúc để hệ thống tạo kết nối và vào Manager để kiểm tra lại.
