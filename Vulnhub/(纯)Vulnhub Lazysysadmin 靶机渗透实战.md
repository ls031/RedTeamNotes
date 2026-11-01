
## 靶机描述

_这是一台标榜难度为简单的开源靶机。作者在描述中说，希望这台靶机们能够教会新手进行 Linux枚举。_

![des](../vulnhubScreenShot/(纯)Vulnhub%20Lazysysadmin%20靶机渗透实战/IMG-20261101195429807.png)


## 信息收集

1. 端口扫描

我们使用端口扫描，发现目标开放了许多端口 22，80，139, 445,3306,6667 。综合来看， 139和 445 的优先级是最高的。因为这两个端口对应着 smb 服务。如果管理员校验不严，这个两个端口极可能暴露一些敏感信息。

```
# Nmap 7.99 scan initiated Sat Oct 31 15:12:37 2026 as: /usr/lib/nmap/nmap --min-rate 10000 -p- -sT -oA nmapscan/TCPS 192.168.1.15
Nmap scan report for 192.168.1.15
Host is up (0.0022s latency).
Not shown: 65529 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3306/tcp open  mysql
6667/tcp open  irc
MAC Address: 00:0C:29:DF:FC:2A (VMware)

# Nmap done at Sat Oct 31 15:12:44 2026 -- 1 IP address (1 host up) scanned in 6.12 seconds
```

2. 详细信息扫描

详细信息扫描暴露出了更为详尽的信息。脚本提示我们当前 Samba 服务版本是 3.X 到 4.X。

```
# Nmap 7.99 scan initiated Sat Oct 31 15:14:51 2026 as: /usr/lib/nmap/nmap -sT -sV -O -sC -p22,80,139,445,3306,6667 -oA nmapscan/details 192.168.1.15
Nmap scan report for 192.168.1.15
Host is up (0.00048s latency).

PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 b5:38:66:0f:a1:ee:cd:41:69:3b:82:cf:ad:a1:f7:13 (DSA)
|   2048 58:5a:63:69:d0:da:dd:51:cc:c1:6e:00:fd:7e:61:d0 (RSA)
|   256 61:30:f3:55:1a:0d:de:c8:6a:59:5b:c9:9c:b4:92:04 (ECDSA)
|_  256 1f:65:c0:dd:15:e6:e4:21:f2:c1:9b:a3:b6:55:a0:45 (ED25519)
80/tcp   open  http        Apache httpd 2.4.7 ((Ubuntu))
|_http-generator: Silex v2.2.7
|_http-server-header: Apache/2.4.7 (Ubuntu)
| http-robots.txt: 4 disallowed entries 
|_/old/ /test/ /TR2/ /Backnode_files/
|_http-title: Backnode
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
3306/tcp open  mysql       MySQL (unauthorized)
6667/tcp open  irc         InspIRCd
| irc-info: 
|   server: Admin.local
|   users: 1
|   servers: 1
|   chans: 0
|   lusers: 1
|   lservers: 0
|   source ident: nmap
|   source host: 192.168.1.4
|_  error: Closing link: (nmap@192.168.1.4) [Client exited]
MAC Address: 00:0C:29:DF:FC:2A (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14
Network Distance: 1 hop
Service Info: Hosts: LAZYSYSADMIN, Admin.local; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_nbstat: NetBIOS name: LAZYSYSADMIN, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: lazysysadmin
|   NetBIOS computer name: LAZYSYSADMIN\x00
|   Domain name: \x00
|   FQDN: lazysysadmin
|_  System time: 2026-10-30T22:40:27+10:00
| smb2-time: 
|   date: 2026-10-30T12:40:28
|_  start_date: N/A
|_clock-skew: mean: -21h54m37s, deviation: 5h46m24s, median: -18h34m37s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Oct 31 15:15:14 2026 -- 1 IP address (1 host up) scanned in 23.52 seconds
```


3. 漏洞脚本扫描

脚本给出了许多可利用点，但是我们先不考虑，如果 samba 没有给出我们特别感兴趣的信息，我们再来考虑漏洞脚本扫描出的信息。

```
# Nmap 7.99 scan initiated Sat Oct 31 15:20:06 2026 as: /usr/lib/nmap/nmap --script=vuln -p22,80,139,445,3306,6667 -oA nmapscan/vulns 192.168.1.15
Nmap scan report for 192.168.1.15
Host is up (0.00060s latency).

PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
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
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-sql-injection: ERROR: Script execution failed (use -d to debug)
| http-enum: 
|   /wordpress/: Blog
|   /test/: Test page
|   /robots.txt: Robots file
|   /info.php: Possible information file
|   /phpmyadmin/: phpMyAdmin
|   /wordpress/wp-login.php: Wordpress login page.
|   /apache/: Potentially interesting directory w/ listing on 'apache/2.4.7 (ubuntu)'
|_  /old/: Potentially interesting directory w/ listing on 'apache/2.4.7 (ubuntu)'
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3306/tcp open  mysql
6667/tcp open  irc
| irc-botnet-channels: 
|_  ERROR: TIMEOUT
|_irc-unrealircd-backdoor: Server closed connection, possibly due to too many reconnects. Try again with argument irc-unrealircd-backdoor.wait set to 100 (or higher if you get this message again).
MAC Address: 00:0C:29:DF:FC:2A (VMware)

Host script results:
|_smb-vuln-ms10-061: false
|_smb-vuln-ms10-054: false
| smb-vuln-regsvc-dos: 
|   VULNERABLE:
|   Service regsvc in Microsoft Windows systems vulnerable to denial of service
|     State: VULNERABLE
|       The service regsvc in Microsoft Windows 2000 systems is vulnerable to denial of service caused by a null deference
|       pointer. This script will crash the service if it is vulnerable. This vulnerability was discovered by Ron Bowes
|       while working on smb-enum-sessions.
|_          

# Nmap done at Sat Oct 31 15:25:28 2026 -- 1 IP address (1 host up) scanned in 321.81 seconds
```


## Samba 服务信息泄露

我们使用 smbclient 工具列举目标开发的所有共享目录。其中这个 share$ 引起了我们的兴趣，看 comment 的描述，这是所以共享文件的汇总。

![p1](../vulnhubScreenShot/(纯)Vulnhub%20Lazysysadmin%20靶机渗透实战/IMG-20261101201027457.png)

其中泄露了许多文件信息，其中 deets.txt 文件我们非常感兴趣。我们使用 get 参数下载到本地。

![p2](../vulnhubScreenShot/(纯)Vulnhub%20Lazysysadmin%20靶机渗透实战/IMG-20261101201218188.png)

![p3](../vulnhubScreenShot/(纯)Vulnhub%20Lazysysadmin%20靶机渗透实战/IMG-20261101201407841.png)

deets.txt 文件中提到了密码 123456。这是很有价值的，但是文件中没有说明用户名。我们还需要进一步信息收集。

## WEB 信息收集

在 WEB 目录翻找时候，我们发现 wordpress 网页路径下，有许多串字符。

```text
My name is togie.
```

![p4](../vulnhubScreenShot/(纯)Vulnhub%20Lazysysadmin%20靶机渗透实战/IMG-20261101204437924.png)


我们猜想这个 togie 可能是 用户名。

## ssh 立足点获取

我们凭借着这个凭据成功登录 ssh 服务器。

```
togie:12345
```


![p5](../vulnhubScreenShot/(纯)Vulnhub%20Lazysysadmin%20靶机渗透实战/IMG-20261101204750840.png)


经过权限枚举，我们发现 togie 这个用户有完整的 sudo 权限。

![p6](../vulnhubScreenShot/(纯)Vulnhub%20Lazysysadmin%20靶机渗透实战/IMG-20261101204951199.png)

执行命令 sudo -i。我们成功获得了 root 权限。


## 总结

这是一条非常简单的靶机，首先我们进行端口扫描，发现目标开放了 139，445。这个端口的价值非常高，我们成功通过 smb 服务泄露文件 deet.txt 文件获得了 管理员泄露的密码 12345。通过 80 端口 ，我们发现 /wordpress/ 网页路径下， 泄露的信息 togie。使用泄露的凭据登录 ssh 服务器。通过简单的枚举，发现具有完整的 sudo 权限，最终提权到 root。