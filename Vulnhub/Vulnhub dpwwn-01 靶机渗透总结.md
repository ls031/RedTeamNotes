
## 靶机描述

_这是一台标榜难度为简单的开源靶机。作者描述，我们的目标是获取 root 权限，并且获得在 /root 目录下面的 dpwwn-01-FLAG.txt 文件。_

![des](../vulnhubScreenShot/Vulnhub%20dpwwn-01%20靶机渗透总结/IMG-20261101205744120.png)


## 信息收集

1. 端口扫描

我们发现目标开放了 22，80，3306 端口。我们并不清楚 mysql 的凭据信息。因此，我们优先尝试渗透 80 端口。

```
# Nmap 7.99 scan initiated Fri Oct 30 16:27:53 2026 as: /usr/lib/nmap/nmap --min-rate 10000 -p- -sT -oA nmapscan/TCPS 192.168.1.14
Nmap scan report for 192.168.1.14
Host is up (0.0010s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
MAC Address: 00:0C:29:48:59:B7 (VMware)

# Nmap done at Fri Oct 30 16:27:55 2026 -- 1 IP address (1 host up) scanned in 2.69 seconds
```

2. 详细信息扫描

详细信息扫描，没有返回有价值的信息。

```
# Nmap 7.99 scan initiated Fri Oct 30 16:28:34 2026 as: /usr/lib/nmap/nmap -sT -sV -O -sC -p22,80,3306 -oA nmapscan/details 192.168.1.14
Nmap scan report for 192.168.1.14
Host is up (0.00055s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 c1:d3:be:39:42:9d:5c:b4:95:2c:5b:2e:20:59:0e:3a (RSA)
|   256 43:4a:c6:10:e7:17:7d:a0:c0:c3:76:88:1d:43:a1:8c (ECDSA)
|_  256 0e:cc:e3:e1:f7:87:73:a1:03:47:b9:e2:cf:1c:93:15 (ED25519)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-title: Apache HTTP Server Test Page powered by CentOS
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
3306/tcp open  mysql   MariaDB 5.5.60
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.60-MariaDB
|   Thread ID: 7
|   Capabilities flags: 63487
|   Some Capabilities: Support41Auth, DontAllowDatabaseTableColumn, SupportsTransactions, IgnoreSigpipes, ConnectWithDatabase, ODBCClient, InteractiveClient, SupportsCompression, FoundRows, Speaks41ProtocolNew, LongColumnFlag, LongPassword, IgnoreSpaceBeforeParenthesis, SupportsLoadDataLocal, Speaks41ProtocolOld, SupportsMultipleResults, SupportsAuthPlugins, SupportsMultipleStatments
|   Status: Autocommit
|   Salt: OlT?!"(AOHZl3X?r@+TD
|_  Auth Plugin Name: mysql_native_password
MAC Address: 00:0C:29:48:59:B7 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14
Network Distance: 1 hop

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Oct 30 16:28:44 2026 -- 1 IP address (1 host up) scanned in 9.98 seconds

```

3. 漏洞脚本扫描

漏洞脚本扫描，显示了 WEB 界面有两个令人感兴趣的网页路径。

```
# Nmap 7.99 scan initiated Fri Oct 30 16:31:09 2026 as: /usr/lib/nmap/nmap --script=vuln -p22,80,3306 -oA nmapscan/vulns 192.168.1.14
Nmap scan report for 192.168.1.14
Host is up (0.00051s latency).

PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-trace: TRACE is enabled
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-enum: 
|   /info.php: Possible information file
|_  /icons/: Potentially interesting folder w/ directory listing
3306/tcp open  mysql
MAC Address: 00:0C:29:48:59:B7 (VMware)

# Nmap done at Fri Oct 30 16:31:41 2026 -- 1 IP address (1 host up) scanned in 32.46 seconds

```


## WEB 渗透

首页是 Apache 正常运行的界面。对我们而言没有任何可以利用的信息。接下来，我们访问 info.php 这个网页路径。

![p1](../vulnhubScreenShot/Vulnhub%20dpwwn-01%20靶机渗透总结/IMG-20261101210637163.png)

info 界面显示的是 phpinfo 函数的信息。这些信息在绕过 waf 的时候极具价值。

![p2](../vulnhubScreenShot/Vulnhub%20dpwwn-01%20靶机渗透总结/IMG-20261101210810830.png)


经过长时间的尝试，我们依旧没有获取立足点。。。。。


## MySQL 未授权登录

出乎意料的是， MySQL 的端口竟然执行 root 用户免密登录。这下子难度降了下来。注意，如果在第一次nmap 扫描时后，发现 MySQL，可以先尝试免密登录。否则如果进行第二次 nmap TCP 扫描后，3306 端口既有可能被锁住，从而错失攻击面。

![p3](../vulnhubScreenShot/Vulnhub%20dpwwn-01%20靶机渗透总结/IMG-20261101211310572.png)

![p4](../vulnhubScreenShot/Vulnhub%20dpwwn-01%20靶机渗透总结/IMG-20261101211520298.png)

最终我们获得了 mistic 用户的凭据。

## ssh 获得立足点

这里的 mistic 用户并没有 sudo 的权限。

![p5](../vulnhubScreenShot/Vulnhub%20dpwwn-01%20靶机渗透总结/IMG-20261101211608671.png)

![p6](../vulnhubScreenShot/Vulnhub%20dpwwn-01%20靶机渗透总结/IMG-20261101211647302.png)

我们发现当前服务器存在定时任务，我们可以修改定时任务，从而提权。

![p7](../vulnhubScreenShot/Vulnhub%20dpwwn-01%20靶机渗透总结/IMG-20261101211751269.png)

大约等待 3分钟左右，我们可以在 /tmp 文件夹下获取我们的 rootbash 文件。最终成功提权。

![p8](../vulnhubScreenShot/Vulnhub%20dpwwn-01%20靶机渗透总结/IMG-20261101211849364.png)


## 总结

这是一台看似简单，其实很考察信息收集功底的机器。如果不尝试 3306 端口，我们还是在 80 端口的死胡同中乱撞。总而言之，要站在全局视角，不要一上来就陷入细节中。