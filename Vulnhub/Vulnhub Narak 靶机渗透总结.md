
# 靶机介绍

_这是一台标榜难度为简单的开源靶机。这是一台以印度宗教为主题的靶机。作者在描述中告诉我们，有两个 flag 等待我们获取。我们需要使用枚举的方式去逃离"地狱"。_

![des](IMG-20260819090455034.png)

# 信息收集

1. **主机发现**
当前kali 的 ip 是 10.10.10.117。目标靶机的 ip 是 10.10.10.119。

2. **端口扫描**

```text
# Nmap 7.99 scan initiated Tue Aug 18 17:09:39 2026 as: /usr/lib/nmap/nmap --min-rate 10000 -p- -oA nmapscan/TCPS 10.10.10.119
Nmap scan report for 10.10.10.119
Host is up (0.00056s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 00:0C:29:B3:F2:59 (VMware)

# Nmap done at Tue Aug 18 17:09:40 2026 -- 1 IP address (1 host up) scanned in 1.53 seconds
```

目标开放了 22，80 端口。我们使用详细脚本扫描查看具体运行的版本信息。

```text
# Nmap 7.99 scan initiated Tue Aug 18 15:53:12 2026 as: /usr/lib/nmap/nmap -sT -sC -sV -O -p22,80 -oA nmapscan/details 10.10.10.119
Nmap scan report for 10.10.10.119
Host is up (0.00070s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 71:bd:59:2d:22:1e:b3:6b:4f:06:bf:83:e1:cc:92:43 (RSA)
|   256 f8:ec:45:84:7f:29:33:b2:8d:fc:7d:07:28:93:31:b0 (ECDSA)
|_  256 d0:94:36:96:04:80:33:10:40:68:32:21:cb:ae:68:f9 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: HA: NARAK
MAC Address: 00:0C:29:B3:F2:59 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Aug 18 15:53:20 2026 -- 1 IP address (1 host up) scanned in 8.56 seconds
```

脚本结果列举出了服务器内核的版本和网站中间件的版本。
我们随后进行了 UDP 端口的扫描。

```text
# Nmap 7.99 scan initiated Tue Aug 18 17:18:41 2026 as: /usr/lib/nmap/nmap -sU --top-ports 100 -oA nmapscan/UDPS 10.10.10.119
Nmap scan report for 10.10.10.119
Host is up (0.00090s latency).
Not shown: 98 closed udp ports (port-unreach)
PORT   STATE         SERVICE
68/udp open|filtered dhcpc
69/udp open|filtered tftp
MAC Address: 00:0C:29:B3:F2:59 (VMware)

# Nmap done at Tue Aug 18 17:20:25 2026 -- 1 IP address (1 host up) scanned in 104.01 seconds
```

可以看到目标的 68，69 可能是开启，也可能是被防火墙限制。

3. **漏洞脚本扫描**

```text
# Nmap 7.99 scan initiated Tue Aug 18 15:53:55 2026 as: /usr/lib/nmap/nmap --script=vuln -p22,80 -oA nmapscan/vulns 10.10.10.119
Nmap scan report for 10.10.10.119
Host is up (0.00074s latency).

PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
| http-internal-ip-disclosure: 
|_  Internal IP Leaked: 127.0.0.1
| http-csrf: 
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=10.10.10.119
|   Found the following possible CSRF vulnerabilities: 
|     
|     Path: http://10.10.10.119:80/
|     Form id: 
|_    Form action: images/666.jpg
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-enum: 
|   /images/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
|_  /webdav/: Potentially interesting folder (401 Unauthorized)
MAC Address: 00:0C:29:B3:F2:59 (VMware)

# Nmap done at Tue Aug 18 15:54:26 2026 -- 1 IP address (1 host up) scanned in 31.33 seconds
```

我们从漏洞脚本扫描的结果中，发现了目标开放了 webdav 服务。有 /webdav 和 /images 两个令我们感兴趣的路径信息。


# WEB 渗透

打开后发现，是一项描述印度宗教的素材图片。根据网站给出的信息，这个 narak 是一个类似地狱的地方。而 yama 则是 28 个神明里面的死神。里面的 yamdoot 是死神的信使。我们仅从网站中了解到这么多信息。

![](IMG-20260819092557044.png)

![](IMG-20260819092951442.png)

没有太多可以利用的地方。我们随即进行网站目录爆破。我们发现有一个 tips.txt 的文件。

![](IMG-20260819093154679.png)

里面暗示我们要打开地狱之门，首先要找到 creds.txt 这样的文件。里面可能存在登录凭据。Webdav 我们没有凭据，没法直接登录。剩下的可能就是从网站给出的图片入手，查看是否存在隐写信息。

![](IMG-20260819093307120.png)

一共 20 张图片。我都下载到本地。使用 exiftool 和 binwalk 工具。都没有发现隐写信息。渗透陷入了僵局。

![](IMG-20260819093902987.png)

其实，前边端口扫描中，我们忽略了一个有价值的端口。就是 69 端口。运行 tftp 服务。这个服务有个特点，就是不能列举所有当前存在的文件。用户需要知道文件名才能获取。这个服务运行在大多物联网设备。也包括虚拟机和物理机的文件读取功能块。这个服务设计的目的就是为了方便稳定读取。我们这边猜测是否，creds.txt 文件就存储在这个 tftp 服务中。
使用这条命令就可以登录 tftp 服务。

```shell
tftp 10.10.10.119
```

![](IMG-20260819094508338.png)

我们获得了 yamdoot:Swarg 这个凭据，发现这不是 ssh 的凭据。那就只有可能是 Webdav 服务的凭据。我们先使用 davtest 工具看是否能上传运行脚本文件。

![](IMG-20260819095818607.png)

通过 cadaver 工具，我们与 Webdav 服务进行交互。上传我们的 php 木马。

![](IMG-20260819100021981.png)

![](IMG-20260819100054664.png)

在 web 端触发我们的 1.php 脚本。拿到 www-data 的权限立足点。

![](IMG-20260819100300905.png)

# 提升立足点

```shell
find / -writable ! -path "/proc/*" ! -path "/sys/*" 2>/dev/null
```

![](IMG-20260819100713120.png)

我们发现一个 hell.sh 的脚本。

![beef](IMG-20260819101212541.png)

其中有一串 brainfuck 编码。通过解密这串编码我们发现了一个口令。

![](IMG-20260819101516512.png)

这很可能是某个用户的登录密码。

![](IMG-20260819101705832.png)

经过尝试，成功登录 inferno 这个用户，获得了第一个 user.txt 的 flag。

![user](IMG-20260819101752094.png)

