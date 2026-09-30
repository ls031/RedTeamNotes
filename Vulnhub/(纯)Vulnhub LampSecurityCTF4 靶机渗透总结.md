
### 靶机描述

 _这是一台标榜难度为简单的 vulnhub 开源靶机。作者说用于基准测试。_

![des](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF4%20靶机渗透总结/IMG-20260930195959208.png)


### 信息收集

1. 端口扫描

我们使用 TCP 协议进行端口扫描。

```text
# Nmap 7.99 scan initiated Tue Sep 29 17:32:04 2026 as: /usr/lib/nmap/nmap --min-rate 10000 -p- -sT -oA nmapscan/TCPS 10.10.10.179
Nmap scan report for 10.10.10.179
Host is up (0.00053s latency).
Not shown: 65512 filtered tcp ports (no-response), 19 filtered tcp ports (host-unreach)
PORT    STATE  SERVICE
22/tcp  open   ssh
25/tcp  open   smtp
80/tcp  open   http
631/tcp closed ipp
MAC Address: 00:0C:29:93:01:1B (VMware)

# Nmap done at Tue Sep 29 17:32:18 2026 -- 1 IP address (1 host up) scanned in 13.88 seconds
```

使用 UDP 协议进行端口扫描

```text
# Nmap 7.99 scan initiated Tue Sep 29 17:35:14 2026 as: /usr/lib/nmap/nmap -sU --top-ports 100 -oA nmapscan/UDPS 10.10.10.179
Nmap scan report for 10.10.10.179
Host is up (0.00047s latency).
All 100 scanned ports on 10.10.10.179 are in ignored states.
Not shown: 60 filtered udp ports (host-prohibited), 40 open|filtered udp ports (no-response)
MAC Address: 00:0C:29:93:01:1B (VMware)

# Nmap done at Tue Sep 29 17:36:11 2026 -- 1 IP address (1 host up) scanned in 57.04 seconds

```

2. 详细信息扫描
```text
# Nmap 7.99 scan initiated Tue Sep 29 17:38:36 2026 as: /usr/lib/nmap/nmap -sT -sC -sV -O -p22,25,80 -oA nmapscan/details 10.10.10.179
Nmap scan report for 10.10.10.179
Host is up (0.00055s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey: 
|   1024 10:4a:18:f8:97:e0:72:27:b5:a4:33:93:3d:aa:9d:ef (DSA)
|_  2048 e7:70:d3:81:00:41:b8:6e:fd:31:ae:0e:00:ea:5c:b4 (RSA)
25/tcp open  smtp    Sendmail 8.13.5/8.13.5
| smtp-commands: ctf4.sas.upenn.edu Hello [10.10.10.117], pleased to meet you, ENHANCEDSTATUSCODES, PIPELINING, EXPN, VERB, 8BITMIME, SIZE, DSN, ETRN, DELIVERBY, HELP
|_ 2.0.0 This is sendmail version 8.13.5 2.0.0 Topics: 2.0.0 HELO EHLO MAIL RCPT DATA 2.0.0 RSET NOOP QUIT HELP VRFY 2.0.0 EXPN VERB ETRN DSN AUTH 2.0.0 STARTTLS 2.0.0 For more info use "HELP <topic>". 2.0.0 To report bugs in the implementation send email to 2.0.0 sendmail-bugs@sendmail.org. 2.0.0 For local information send email to Postmaster at your site. 2.0.0 End of HELP info
80/tcp open  http    Apache httpd 2.2.0 ((Fedora))
|_http-title:  Prof. Ehks 
|_http-server-header: Apache/2.2.0 (Fedora)
| http-robots.txt: 5 disallowed entries 
|_/mail/ /restricted/ /conf/ /sql/ /admin/
MAC Address: 00:0C:29:93:01:1B (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|proxy server|remote management|WAP|webcam|terminal server
Running (JUST GUESSING): Linux 2.6.X (97%), SonicWALL embedded (92%), Dell iDRAC 6 (91%), Asus embedded (91%), Control4 embedded (90%), Mobotix embedded (90%), Lantronix embedded (90%)
OS CPE: cpe:/o:linux:linux_kernel:2.6 cpe:/o:sonicwall:aventail_ex-6000 cpe:/o:dell:idrac6_firmware cpe:/o:linux:linux_kernel:2.6.22 cpe:/h:asus:rt-n16 cpe:/h:lantronix:slc_8
Aggressive OS guesses: Linux 2.6.16 - 2.6.21 (97%), Linux 2.6.8 - 2.6.30 (97%), Linux 2.6.23 (93%), Linux 2.6.13 - 2.6.32 (93%), Linux 2.6.16 (93%), SonicWALL Aventail EX-6000 VPN appliance (92%), Linux 2.6.23 - 2.6.38 (91%), Linux 2.6.9 - 2.6.18 (91%), Dell iDRAC 6 remote access controller (Linux 2.6) (91%), Linux 2.6.16 - 2.6.25 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop
Service Info: Host: ctf4.sas.upenn.edu; OS: Unix

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Sep 29 17:38:47 2026 -- 1 IP address (1 host up) scanned in 11.11 seconds

```

默认脚本帮我枚举出了网站几个感兴趣的文件目录。

3. 漏洞脚本扫描

漏洞脚本指出当前网站存在许多sql 注入漏洞的。

```text
# Nmap 7.99 scan initiated Tue Sep 29 17:45:21 2026 as: /usr/lib/nmap/nmap --script=vuln -p22,25,80 -oA nmapscan/vulns 10.10.10.179
Nmap scan report for 10.10.10.179
Host is up (0.00050s latency).

PORT   STATE SERVICE
22/tcp open  ssh
25/tcp open  smtp
| smtp-vuln-cve2010-4344: 
|_  The SMTP server is not Exim: NOT VULNERABLE
80/tcp open  http
|_http-trace: TRACE is enabled
|_http-sql-injection: ERROR: Script execution failed (use -d to debug)
| http-slowloris-check: 
|   VULNERABLE:
|   Slowloris DOS attack
|     State: LIKELY VULNERABLE
|     IDs:  CVE:CVE-2007-6750
|       Slowloris tries to keep many connections to the target web server open and hold
|       them open as long as possible.  It accomplishes this by opening connections to
|       the target web server and sending a partial request. By doing so, it starves
|       the http server's resources causing Denial Of Service.
|       
|     Disclosure date: 2009-09-17
|     References:
|       http://ha.ckers.org/slowloris/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-6750
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-csrf: 
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=10.10.10.179
|   Found the following possible CSRF vulnerabilities: 
|     
|     Path: http://10.10.10.179:80/
|     Form id: 
|     Form action: /index.html?page=search&title=Search Results
|     
|     Path: http://10.10.10.179:80/index.html?page=contact&title=Contact
|     Form id: 
|     Form action: /index.html?page=search&title=Search Results
|     
|     Path: http://10.10.10.179:80/index.html?title=Home Page
|     Form id: 
|     Form action: /index.html?page=search&title=Search Results
|     
|     Path: http://10.10.10.179:80/index.html?page=research&title=Research
|     Form id: 
|     Form action: /index.html?page=search&title=Search Results
|     
|     Path: http://10.10.10.179:80/index.html?page=blog&title=Blog
|     Form id: 
|     Form action: /index.html?page=search&title=Search Results
|     
|     Path: http://10.10.10.179:80/index.html?page=search&title=Search Results
|     Form id: 
|     Form action: /index.html?page=search&title=Search Results
|     
|     Path: http://10.10.10.179:80/?page=blog&title=Blog&id=5
|     Form id: 
|     Form action: /index.html?page=search&title=Search Results
|     
|     Path: http://10.10.10.179:80/?page=blog&title=Blog&id=6
|     Form id: 
|     Form action: /index.html?page=search&title=Search Results
|     
|     Path: http://10.10.10.179:80/?page=blog&title=Blog&id=2
|     Form id: 
|     Form action: /index.html?page=search&title=Search Results
|     
|     Path: http://10.10.10.179:80/?page=blog&title=Blog&id=7
|     Form id: 
|_    Form action: /index.html?page=search&title=Search Results
| http-enum: 
|   /admin/: Possible admin folder
|   /admin/index.php: Possible admin folder
|   /admin/login.php: Possible admin folder
|   /admin/admin.php: Possible admin folder
|   /robots.txt: Robots file
|   /icons/: Potentially interesting directory w/ listing on 'apache/2.2.0 (fedora)'
|   /images/: Potentially interesting directory w/ listing on 'apache/2.2.0 (fedora)'
|   /inc/: Potentially interesting directory w/ listing on 'apache/2.2.0 (fedora)'
|   /pages/: Potentially interesting directory w/ listing on 'apache/2.2.0 (fedora)'
|   /restricted/: Potentially interesting folder (401 Authorization Required)
|   /sql/: Potentially interesting directory w/ listing on 'apache/2.2.0 (fedora)'
|_  /usage/: Potentially interesting folder
MAC Address: 00:0C:29:93:01:1B (VMware)

# Nmap done at Tue Sep 29 17:47:44 2026 -- 1 IP address (1 host up) scanned in 142.94 seconds

```


### WEB 渗透

### 任意文件读取漏洞

在测试过程中，发现网站存在一个文件包含参数 page。经过测试，可以通过 %00 截断的方式截去后缀。使用 ../../../../../ 等方式绕过前面的限制。

![LFI](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF4%20靶机渗透总结/IMG-20260930211107945.png)
但是使用 php filter 依旧读取不了文件内容。没法通过这种方式获取 blog.php 的源码。
后面通过 ssh 立足点我查看到了源码。

![LFI1](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF4%20靶机渗透总结/IMG-20260930212055429.png)

这是属于文件前面不可控，文件后面可控的情况。这种情况是没法使用 Wrapper，也就是使用不了php 伪协议。而且服务器是使用 include 函数这种文件包含的函数，我们没法读取 php 代码。

#### SQLmap 获得登录凭据
登录 url 处发现字段 id 存在数字型注入。使用 _id=6_ 和 _id=7-1_ 返回的内容是一致的。但是这个网站并没有给我透露太多的报错信息。所以，目前可以采取盲注和延时注入的技巧。为了方便操作，我们使用 SQLmap 工具来自动化操作。


![p1](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF4%20靶机渗透总结/IMG-20260930201032749.png)

![p2](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF4%20靶机渗透总结/IMG-20260930201055342.png)


![p3](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF4%20靶机渗透总结/IMG-20260930201641943.png)


在信息收集的过程中，我们发现了网站创建的 sql 文件。不过不能采用 union 联合注入的方式攻击，这个文件价值不大。

![p4](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF4%20靶机渗透总结/IMG-20260930201601115.png)

SQLmap 也检测出存在逻辑盲注和延时注入。

![p5](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF4%20靶机渗透总结/IMG-20260930201901604.png)

![p6](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF4%20靶机渗透总结/IMG-20260930201924447.png)


我们获得的凭据
```text
achen           seventysixers
dstevens        ilike2surf
ghighland       undone1
jdurbin         Sue1978
pmoore          Homesite
sorzek          pacman
```


### SSH 凭据碰撞获得立足点

在进行凭据碰撞时，发现 KALI 的 ssh 版本太高了。默认不使用旧版的密钥交换和加密算法。我们需要手动指定过程参数。

![p7](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF4%20靶机渗透总结/IMG-20260930202616775.png)

其实其他账号也可以，
![p8](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF4%20靶机渗透总结/IMG-20260930210631476.png)


我们获得立足点后，发现我们能够使用 sudo 的所以权限。成功提权到了 root。


### 总结

整体上来说，这台靶机还是比较简单的。漏洞利用的过程也很明显。