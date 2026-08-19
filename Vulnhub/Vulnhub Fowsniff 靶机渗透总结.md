# 靶机描述

_这是一台标榜难度为简单的开源靶机。作者说这是一台 boot2root 类型的靶机，我们需要通过自身利用搜索引擎的技巧来攻克这台靶机。另外作者还说，这台靶机如果运行在 VMware 上面需要调整一下配置。我们需要通过 linux 单用户模式来调整靶机的网卡名。_

![des](../vulnhubScreenShot/Vulnhub%20Fowsniff%20靶机渗透总结/IMG-20260819132524298.png)

# 信息收集

1. **主机发现**
我们本地 kali 的 ip 是 10.10.10.117，目标靶机的 ip 是 10.10.10.118。

2. **端口扫描**

```text
# Nmap 7.99 scan initiated Tue Aug 18 09:25:34 2026 as: /usr/lib/nmap/nmap --min-rate 10000 -p- -oA nmapscan/TCPS 10.10.10.118
Nmap scan report for 10.10.10.118
Host is up (0.0010s latency).
Not shown: 65531 closed tcp ports (reset)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
110/tcp open  pop3
143/tcp open  imap
MAC Address: 00:0C:29:38:41:52 (VMware)

# Nmap done at Tue Aug 18 09:25:39 2026 -- 1 IP address (1 host up) scanned in 5.41 seconds
```

可见目标开放了挺多的端口。接下来，我们对目标的详细信息进行扫描。

```text
# Nmap 7.99 scan initiated Tue Aug 18 09:30:33 2026 as: /usr/lib/nmap/nmap -sT -sC -sV -O -p22,80,110,143 -oA nmapscan/details 10.10.10.118
Nmap scan report for 10.10.10.118
Host is up (0.00098s latency).

PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 90:35:66:f4:c6:d2:95:12:1b:e8:cd:de:aa:4e:03:23 (RSA)
|   256 53:9d:23:67:34:cf:0a:d5:5a:9a:11:74:bd:fd:de:71 (ECDSA)
|_  256 a2:8f:db:ae:9e:3d:c9:e6:a9:ca:03:b1:d7:1b:66:83 (ED25519)
80/tcp  open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Fowsniff Corp - Delivering Solutions
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-robots.txt: 1 disallowed entry 
|_/
110/tcp open  pop3    Dovecot pop3d
|_pop3-capabilities: SASL(PLAIN) TOP USER RESP-CODES UIDL AUTH-RESP-CODE CAPA PIPELINING
143/tcp open  imap    Dovecot imapd
|_imap-capabilities: more have post-login listed capabilities LOGIN-REFERRALS Pre-login LITERAL+ ENABLE ID IDLE AUTH=PLAINA0001 IMAP4rev1 OK SASL-IR
MAC Address: 00:0C:29:38:41:52 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.2 - 4.14 (98%), Linux 3.10 - 4.11 (95%), Linux 3.18 (95%), OpenWrt Chaos Calmer 15.05 (Linux 3.18) or Designated Driver (Linux 4.1 or 4.4) (95%), Sony Android TV (Android 5.0) (95%), Android 5.1 (95%), Android 7.1.2 (Linux 3.4) (95%), Linux 3.2 - 3.16 (95%), Linux 4.4 (95%), Linux 3.8 - 3.16 (94%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Aug 18 09:30:44 2026 -- 1 IP address (1 host up) scanned in 10.91 seconds
```

脚本对目标服务器的操作系统内核版本给出了模糊的说明，我们暂时认定这是一个 linux 3.2 内核的系统。pop3 和 imap 服务也没有泄露敏感信息。

```text
# Nmap 7.99 scan initiated Tue Aug 18 09:26:53 2026 as: /usr/lib/nmap/nmap -sU --min-rate 10000 -p- -oA nmapscan/UDPS 10.10.10.118
Warning: 10.10.10.118 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.10.118
Host is up (0.00084s latency).
All 65535 scanned ports on 10.10.10.118 are in ignored states.
Not shown: 65457 open|filtered udp ports (no-response), 78 closed udp ports (port-unreach)
MAC Address: 00:0C:29:38:41:52 (VMware)

# Nmap done at Tue Aug 18 09:28:07 2026 -- 1 IP address (1 host up) scanned in 73.30 seconds
```

UDP 扫描也没有扫描出感兴趣的端口。我们下面进行漏洞脚本扫描，看看是否有低摘的果子。

3. **漏洞脚本扫描**


```text
# Nmap 7.99 scan initiated Tue Aug 18 09:31:39 2026 as: /usr/lib/nmap/nmap --script=vuln -p22,80,110,143 -oA nmapscan/vulns 10.10.10.118
Nmap scan report for 10.10.10.118
Host is up (0.0011s latency).

PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
| http-internal-ip-disclosure: 
|_  Internal IP Leaked: 127.0.1.1
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
|_http-sql-injection: ERROR: Script execution failed (use -d to debug)
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum: 
|   /robots.txt: Robots file
|   /README.txt: Interesting, a readme.
|_  /images/: Potentially interesting directory w/ listing on 'apache/2.4.18 (ubuntu)'
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
110/tcp open  pop3
143/tcp open  imap
MAC Address: 00:0C:29:38:41:52 (VMware)

# Nmap done at Tue Aug 18 09:37:01 2026 -- 1 IP address (1 host up) scanned in 321.89 seconds
```

漏洞脚本扫描暴露出来一个 /images目录，还有两个文件。我们还是需要结合浏览器来进行分析。

# WEB 渗透

这是一个公司的前台网页。但是描述中说，网站功能停止运行了。网站经过黑客工具，泄露了内部员工的账号密码。但是用户信息没有泄露。我们关注到公告中有一行提到，黑客劫持了公司的推特账户。

![web1](../vulnhubScreenShot/Vulnhub%20Fowsniff%20靶机渗透总结/IMG-20260819134228011.png)

我们在目录爆破过程中，发现了一个 security 文件。里面明目张胆的写了黑客组织的攻击公示。

![web2](../vulnhubScreenShot/Vulnhub%20Fowsniff%20靶机渗透总结/IMG-20260819134951890.png)


我原先以 B1gN1nj4 这个团伙名为关键字进行搜索。结果发现了黑客组织泄露的用户凭据。

![web3](../vulnhubScreenShot/Vulnhub%20Fowsniff%20靶机渗透总结/IMG-20260819135305431.png)

![web4](../vulnhubScreenShot/Vulnhub%20Fowsniff%20靶机渗透总结/IMG-20260819135320735.png)

为了理解这个黑客团体攻击造成的影响。我们还是通过 Google 搜索到了公司的推特账号。内容还是挺不乐观的。结果第一条就是。

![p1](../vulnhubScreenShot/Vulnhub%20Fowsniff%20靶机渗透总结/IMG-20260819140110181.png)

![pwned](../vulnhubScreenShot/Vulnhub%20Fowsniff%20靶机渗透总结/IMG-20260819140154530.png)

这个攻击者直接将密码 dump 出来了。但是由于时间原因，pastebin.com 网站不在存有这个 泄露的密码信息了。搜索评论，发生有人将 dump 出来的信息归档了。也就是我们之前看到的那个文件。

![pass](../vulnhubScreenShot/Vulnhub%20Fowsniff%20靶机渗透总结/IMG-20260819140443697.png)

# 破解凭据，POP3 服务器渗透


这是泄露的 dump 信息文件,攻击者告诉我们这是邮箱的密码凭据。

```text
FOWSNIFF CORP PASSWORD LEAK
            ''~``
           ( o o )
+-----.oooO--(_)--Oooo.------+
|                            |
|          FOWSNIFF          |
|            got             |
|           PWN3D!!!         |
|                            |         
|       .oooO                |         
|        (   )   Oooo.       |         
+---------\ (----(   )-------+
           \_)    ) /
                 (_/
FowSniff Corp got pwn3d by B1gN1nj4!
No one is safe from my 1337 skillz!
 
 
mauer@fowsniff:8a28a94a588a95b80163709ab4313aa4
mustikka@fowsniff:ae1644dac5b77c0cf51e0d26ad6d7e56
tegel@fowsniff:1dc352435fecca338acfd4be10984009
baksteen@fowsniff:19f5af754c31f1e2651edde9250d69bb
seina@fowsniff:90dc16d47114aa13671c697fd506cf26
stone@fowsniff:a92b8a29ef1183192e3d35187e0cfabd
mursten@fowsniff:0e9588cb62f4b6f27e33d449e2ba0b3b
parede@fowsniff:4d6e42f56e127803285a0a7649b5ab11
sciana@fowsniff:f7fd98d380735e859f8b2ffbbede5a7e
 
Fowsniff Corporation Passwords LEAKED!
FOWSNIFF CORP PASSWORD DUMP!
 
Here are their email passwords dumped from their databases.
They left their pop3 server WIDE OPEN, too!
 
MD5 is insecure, so you shouldn't have trouble cracking them but I was too lazy haha =P
 
l8r n00bz!
 
B1gN1nj4

-------------------------------------------------------------------------------------------------
This list is entirely fictional and is part of a Capture the Flag educational challenge.

--- THIS IS NOT A REAL PASSWORD LEAK ---
 
All information contained within is invented solely for this purpose and does not correspond
to any real persons or organizations.
 
Any similarities to actual people or entities is purely coincidental and occurred accidentally.

-------------------------------------------------------------------------------------------------
```

解密后，我们尝试去登录 网站的邮箱，查看是否有可以利用的信息泄露。

```text
mauer@fowsniff:8a28a94a588a95b80163709ab4313aa4(mailcall)
mustikka@fowsniff:ae1644dac5b77c0cf51e0d26ad6d7e56(bilbo101)
tegel@fowsniff:1dc352435fecca338acfd4be10984009(apples01)
baksteen@fowsniff:19f5af754c31f1e2651edde9250d69bb(skyler22)
seina@fowsniff:90dc16d47114aa13671c697fd506cf26(scoobydoo2)
stone@fowsniff:a92b8a29ef1183192e3d35187e0cfabd
mursten@fowsniff:0e9588cb62f4b6f27e33d449e2ba0b3b(carp4ever)
parede@fowsniff:4d6e42f56e127803285a0a7649b5ab11(orlando12)
sciana@fowsniff:f7fd98d380735e859f8b2ffbbede5a7e(07011972)
```

![hydra](../vulnhubScreenShot/Vulnhub%20Fowsniff%20靶机渗透总结/IMG-20260819141415039.png)

我们使用这个凭据登录 pop3 服务器，发现用户的两份邮件。

![pop3](../vulnhubScreenShot/Vulnhub%20Fowsniff%20靶机渗透总结/IMG-20260819141936994.png)

邮件1 的大致信息是，黑客通过数据库语句不严格过滤，突破页面。获得员工的邮箱密码。为了进行隔离。员工需要进一步到新的的服务器上工作，也就是当前目标靶机。这里暴露 ssh 的临时登录口令。

```text
SSH is "S1ck3nBluff+secureshell"
```

通过这个凭据，我们成功获得了立足点。

![stepstone](../vulnhubScreenShot/Vulnhub%20Fowsniff%20靶机渗透总结/IMG-20260819142416917.png)


# 权限提升

通过 find 命令搜索，我们发现 /mnt/cube/cube.sh 这个文件。

![cube](../vulnhubScreenShot/Vulnhub%20Fowsniff%20靶机渗透总结/IMG-20260819142958874.png)

这个是我们刚获得立足点时，展示给我们的图案。这个脚本肯定是给多个用户都发送这样的的信息。所以必定是以 root 权限运行。我们可以在里面写入我们的提权命令。我们就可以获得一个高权限的会话。

![p1](../vulnhubScreenShot/Vulnhub%20Fowsniff%20靶机渗透总结/IMG-20260819143229336.png)

![p2](../vulnhubScreenShot/Vulnhub%20Fowsniff%20靶机渗透总结/IMG-20260819143307479.png)

成功拿到了 root 会话，查看我们的战利品。

![root](../vulnhubScreenShot/Vulnhub%20Fowsniff%20靶机渗透总结/IMG-20260819143431714.png)

# 总结

这是一台很有趣的靶机，我们通过前期的端口扫描，发现目标站点开放了 22，80，110，143端口。通过 80 端口对网站信息的理解，我们找到了泄露的邮箱登录凭据。通过邮箱给出的信息，我们发现了 ssh 的临时登录凭据。成功获得立足点。通过权限枚举发现，一个给所有都发送信息的脚本。猜测有root 权限运行。我们随即写入提权命令，成功获得 root shell，拿下靶机。