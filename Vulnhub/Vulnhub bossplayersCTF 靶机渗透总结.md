
## 靶机描述

这是一台标榜难度为简单的开源靶机，作者甚至在描述中放狠话，我们应该在 30 分钟之内，拿下 root 权限。

![des](../vulnhubScreenShot/Vulnhub%20bossplayersCTF%20靶机渗透总结/IMG-20261030140458057.png)


## 信息收集

1. 端口扫描

```
# Nmap 7.99 scan initiated Sun Oct 25 16:07:52 2026 as: /usr/lib/nmap/nmap --min-rate 10000 -p- -oA nmapscan/TCPS 192.168.1.12
Nmap scan report for 192.168.1.12
Host is up (0.00072s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 00:0C:29:53:FC:4C (VMware)

# Nmap done at Sun Oct 25 16:07:59 2026 -- 1 IP address (1 host up) scanned in 6.99 seconds
```

目标开发了 22，80 端口。一般情况下， 80 端口会暴露出大量的攻击面，而 22 端口如果没有获得凭据，  只能进行暴力破解。我们选择 80 端口为第一优先级。

2. 详细信息扫描

根据信息，我们了解到了目标运行的中间件版本和服务信息。

```
# Nmap 7.99 scan initiated Sun Oct 25 16:07:25 2026 as: /usr/lib/nmap/nmap -sT -sV -O -sC -p22,80 -oA nmapscan/details 192.168.1.12
Nmap scan report for 192.168.1.12
Host is up (0.00079s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10 (protocol 2.0)
| ssh-hostkey: 
|   2048 ac:0d:1e:71:40:ef:6e:65:91:95:8d:1c:13:13:8e:3e (RSA)
|   256 24:9e:27:18:df:a4:78:3b:0d:11:8a:92:72:bd:05:8d (ECDSA)
|_  256 26:32:8d:73:89:05:29:43:8e:a1:13:ba:4f:83:53:f8 (ED25519)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.38 (Debian)
MAC Address: 00:0C:29:53:FC:4C (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Oct 25 16:07:37 2026 -- 1 IP address (1 host up) scanned in 12.68 seconds
```

3. 漏洞脚本扫描

漏洞脚本扫描的结果暴露给我们，目标网站有 logs.php 文件和 robots.txt 文件。这是我们要首先查看的，很可能会泄露出敏感信息。

```
# Nmap 7.99 scan initiated Sun Oct 25 16:08:30 2026 as: /usr/lib/nmap/nmap --script=vuln -p22,80 -oA nmapscan/vulns 192.168.1.12
Nmap scan report for 192.168.1.12
Host is up (0.00069s latency).

PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-enum: 
|   /logs.php: Logs
|_  /robots.txt: Robots file
MAC Address: 00:0C:29:53:FC:4C (VMware)

# Nmap done at Sun Oct 25 16:09:06 2026 -- 1 IP address (1 host up) scanned in 35.44 seconds
```



## WEB 渗透

打开页面，我们发现，在页面的源码中，有一串字符串。对于字符串的识别，我们推荐使用 [hashes](hashes.com) 这个网站，它能更加详细的识别字符串的各种类别。

![p1](../vulnhubScreenShot/Vulnhub%20bossplayersCTF%20靶机渗透总结/IMG-20261030143341546.png)

根据结果，我们发现这是一串经过 base64 编码后的字符串。

![p2](../vulnhubScreenShot/Vulnhub%20bossplayersCTF%20靶机渗透总结/IMG-20261030143955195.png)

![p2-1](../vulnhubScreenShot/Vulnhub%20bossplayersCTF%20靶机渗透总结/IMG-20261030144258443.png)

![p3](../vulnhubScreenShot/Vulnhub%20bossplayersCTF%20靶机渗透总结/IMG-20261030144029764.png)

我得到了一个隐蔽的网站文件路径，workinginprogress.php。查找一下我们的 robots.txt 文件，我们发现，我也发现了一串字符串，但是经过解密后，没有任何可利用的信息。

![p4](../vulnhubScreenShot/Vulnhub%20bossplayersCTF%20靶机渗透总结/IMG-20261030144403119.png)

![p5](../vulnhubScreenShot/Vulnhub%20bossplayersCTF%20靶机渗透总结/IMG-20261030144432322.png)

### FUZZ 工具检查命令执行参数

根据 workinginprogress.php 界面的提示，当前界面可能存在一个可以命令执行的入口点。

![p6](../vulnhubScreenShot/Vulnhub%20bossplayersCTF%20靶机渗透总结/IMG-20261030145340765.png)

我们猜测，这个注入参数肯定是以 GET 或者 POST 方式进行传参的。我们使用 Fuff 工具来猜测这个可能的借口参数。 Fuff 工具会通过指定的字段，在请求访问过程中的指定位置上填写参数名，向服务器发包，如果服务器返回与其他请求不同的结果说明，这个请求的参数名很可能是开发者遗漏的参数。

这是错误的写法，FUZZ 就是我们指定要猜解的参数名。注意等号后面我们不能填写 1 ，最好填写一条可以执行的命令，这样当命令执行成功和执行失败返回的响应包的数据大小是不一样的。

![p7](../vulnhubScreenShot/Vulnhub%20bossplayersCTF%20靶机渗透总结/IMG-20261030145952753.png)

![p8](../vulnhubScreenShot/Vulnhub%20bossplayersCTF%20靶机渗透总结/IMG-20261030150245210.png)


## 权限提升

获得立足点后，我们在枚举具备 SUID 执行权限的文件时候，我们发现 find 命令。我们很是感兴趣。我们可以参考 gtfobins 这个网站来获取提权的命令。

![findpri](../vulnhubScreenShot/Vulnhub%20bossplayersCTF%20靶机渗透总结/IMG-20261030150629374.png)

![p9](../vulnhubScreenShot/Vulnhub%20bossplayersCTF%20靶机渗透总结/IMG-20261030150518255.png)

![p10](../vulnhubScreenShot/Vulnhub%20bossplayersCTF%20靶机渗透总结/IMG-20261030150547035.png)


## 总结

这是一台标准的 CTF 攻击链路机器，经过信息收集，我们发现了目标端口开放了 22，80 端口。根据优先级，我们先通过 80 端口拿信息。我们在前端界面发现了一串 Base64 编码三次的字符串，通过解码，我们找到了具有命令执行漏洞的 workinginprogress.php 的网页路径。通过 fuzz 测试，我们找到了那个具有命令执行漏洞的参数。成功获得 SHELL。然后通过查找 SUID 权限的文件，利用 find 工作进行命令执行，获得 root 权限。
