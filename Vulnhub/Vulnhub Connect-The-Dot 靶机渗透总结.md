
# 靶机描述

_这是一台标榜难度为入门级别的开源靶机。我们需要寻找拿到 user.txt 和 root.txt。_

![des](../vulnhubScreenShot/Vulnhub%20Connect-The-Dot%20靶机渗透总结/IMG-20260826083334878.png)

# 信息收集

1. 主机发现

目标 IP 是 10.10.10.122 。而本地 kali 是 10.10.10.117 。

2. 端口扫描

```text
# Nmap 7.99 scan initiated Wed Aug 26 08:38:29 2026 as: /usr/lib/nmap/nmap -sT --min-rate=10000 -p- -oA nmapscan/TCPS 10.10.10.122
Nmap scan report for 10.10.10.122
Host is up (0.00022s latency).
Not shown: 65526 closed tcp ports (conn-refused)
PORT      STATE SERVICE
21/tcp    open  ftp
80/tcp    open  http
111/tcp   open  rpcbind
2049/tcp  open  nfs
7822/tcp  open  unknown
37399/tcp open  unknown
39153/tcp open  unknown
44697/tcp open  unknown
47795/tcp open  unknown
MAC Address: 00:0C:29:8B:EB:45 (VMware)

# Nmap done at Wed Aug 26 08:38:32 2026 -- 1 IP address (1 host up) scanned in 2.70 seconds
```

TCP 扫描展示了许多端口。为了信息的完整性，我们使用 UDP 进行扫描。

```text
# Nmap 7.99 scan initiated Wed Aug 26 08:39:09 2026 as: /usr/lib/nmap/nmap -sU --top-ports 100 -oA nmapscan/UDPS --reason 10.10.10.122
Nmap scan report for 10.10.10.122
Host is up, received arp-response (0.0011s latency).
Not shown: 57 closed udp ports (port-unreach), 40 open|filtered udp ports (no-response)
PORT     STATE SERVICE  REASON
111/udp  open  rpcbind  udp-response ttl 64
2049/udp open  nfs      udp-response ttl 64
5353/udp open  zeroconf udp-response ttl 255
MAC Address: 00:0C:29:8B:EB:45 (VMware)

# Nmap done at Wed Aug 26 08:40:03 2026 -- 1 IP address (1 host up) scanned in 53.76 seconds
```

UDP 扫描没有给出特别让我们感兴趣的结果。对于 TCP 扫描的结果，我们进行详细信息扫描和版本枚举。

```text
# Nmap 7.99 scan initiated Wed Aug 26 08:43:14 2026 as: /usr/lib/nmap/nmap -sT -sV -O -p21,80,111,2049,7822,37399,39153,44697,47795 -oA nmapscan/details 10.10.10.122
Nmap scan report for 10.10.10.122
Host is up (0.0011s latency).

PORT      STATE SERVICE  VERSION
21/tcp    open  ftp      vsftpd 2.0.8 or later
80/tcp    open  http     Apache httpd 2.4.38 ((Debian))
111/tcp   open  rpcbind  2-4 (RPC #100000)
2049/tcp  open  nfs      3-4 (RPC #100003)
7822/tcp  open  ssh      OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
37399/tcp open  mountd   1-3 (RPC #100005)
39153/tcp open  nlockmgr 1-4 (RPC #100021)
44697/tcp open  mountd   1-3 (RPC #100005)
47795/tcp open  mountd   1-3 (RPC #100005)
MAC Address: 00:0C:29:8B:EB:45 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Aug 26 08:43:27 2026 -- 1 IP address (1 host up) scanned in 13.34 seconds
```

WEB 中间件和 ftp 服务器的版本都比较低。这里的 ssh 服务运行在 7822 端口。其他的都是 nfs 的服务。操作系统版本我们暂且认为它是 Linux 3.X 版本。下面进行漏洞脚本扫描。

# 漏洞脚本扫描

```text
# Nmap 7.99 scan initiated Wed Aug 26 08:54:07 2026 as: /usr/lib/nmap/nmap --script=vuln -p21,80,111,2049,7822,37399,39153,44697,47795 -oA nmapscan/vulns 10.10.10.122
Nmap scan report for 10.10.10.122
Host is up (0.0020s latency).

PORT      STATE SERVICE
21/tcp    open  ftp
80/tcp    open  http
| http-internal-ip-disclosure: 
|_  Internal IP Leaked: 127.0.1.1
|_http-sql-injection: ERROR: Script execution failed (use -d to debug)
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-enum: 
|   /images/: Potentially interesting directory w/ listing on 'apache/2.4.38 (debian)'
|_  /manual/: Potentially interesting folder
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-fileupload-exploiter: 
|   
|     Couldn't find a file-type field.
|   
|     Couldn't find a file-type field.
|   
|_    Couldn't find a file-type field.
111/tcp   open  rpcbind
2049/tcp  open  nfs
7822/tcp  open  unknown
37399/tcp open  unknown
39153/tcp open  unknown
44697/tcp open  unknown
47795/tcp open  unknown
MAC Address: 00:0C:29:8B:EB:45 (VMware)

# Nmap done at Wed Aug 26 08:54:36 2026 -- 1 IP address (1 host up) scanned in 28.66 seconds
```

漏洞脚本扫描结果来看，ftp 不存在匿名用户登录的漏洞。WEB 服务暴露出了两个令我们感兴趣的文件路径，image 和 manual 路径。

# NFS 文件系统挂载

我想探测一下 NFS 服务能否进行远程挂载。
我们可以参考这篇文件的操作方案。

![nfs1](../vulnhubScreenShot/Vulnhub%20Connect-The-Dot%20靶机渗透总结/IMG-20260826090044648.png)

通过在家目录下面搜索，我们发现了 ssh 公私钥。

![nfs2](../vulnhubScreenShot/Vulnhub%20Connect-The-Dot%20靶机渗透总结/IMG-20260826090305748.png)

虽然拥有了私钥，但是我们登录 ssh 依旧需要口令密码。

# WEB 渗透

打开页面后，提示我们 web 站点存在信息值得我们获取。我们查看 web 前端的源代码，发现有个文字性描述。大致意思是说，当前服务面临资源耗尽，不定时的死机的问题。这样和 Morris 的 描述相符合。

![web1](../vulnhubScreenShot/Vulnhub%20Connect-The-Dot%20靶机渗透总结/IMG-20260826090503130.png)

但是有一行注释引起我们的注意，
```text
<!-- Reverse of norris -->
```
这里的 norris 很像是一个登录用户名。我们随即尝试了一下 ftp，结果印证了我们的想法。但是我们已经没有登录密码。

![web2](../vulnhubScreenShot/Vulnhub%20Connect-The-Dot%20靶机渗透总结/IMG-20260826091053525.png)

![web3](../vulnhubScreenShot/Vulnhub%20Connect-The-Dot%20靶机渗透总结/IMG-20260826091446588.png)

## 网页目录爆破扫描

gobuster 的结果中，我们对 hits.txt 这个文件很感兴趣。可能对我们的行动有启发性的提示。

![w1](../vulnhubScreenShot/Vulnhub%20Connect-The-Dot%20靶机渗透总结/IMG-20260826091609358.png)

它提示我们需要保持更强的枚举能力。我访问 mysite 这个网站路径。

![w2](../vulnhubScreenShot/Vulnhub%20Connect-The-Dot%20靶机渗透总结/IMG-20260826091733720.png)

里面有许多文件，我们最感兴趣的是 bootstrap.min.cs 这个文件。

![w3](../vulnhubScreenShot/Vulnhub%20Connect-The-Dot%20靶机渗透总结/IMG-20260826092027718.png)

打开发现这是编码处理过的。

![w4](../vulnhubScreenShot/Vulnhub%20Connect-The-Dot%20靶机渗透总结/IMG-20260826092203164.png)

## jsFuck 解码

经过 Google 搜索 jsfuck。我们了解到 jsfuck 是一种基于 javascript 的编程范式。上述的文件其实就是经过混淆后的 js 代码。

![j1](../vulnhubScreenShot/Vulnhub%20Connect-The-Dot%20靶机渗透总结/IMG-20260826092739307.png)

根据网页中描述说，jsfuck 只通过 6 个不同的字符构成。因此，为了解码上述文件，我们需要去掉变量名。进行解码操作。

![j2](../vulnhubScreenShot/Vulnhub%20Connect-The-Dot%20靶机渗透总结/IMG-20260826093324058.png)

我们获得了 norris 用户的凭据，TryToGuessThisNorris@2k19。
经过尝试，我们获得了 norris 用户的 ssh 和 ftp 服务凭据。

## 莫尔斯密码解密

我们登录 ftp服务器，发现了许多备份文件。

![m1](../vulnhubScreenShot/Vulnhub%20Connect-The-Dot%20靶机渗透总结/IMG-20260826093916243.png)

我们随即下载到本地，其中 game.jpg.bak 文件发现存在注释信息。

![m2](../vulnhubScreenShot/Vulnhub%20Connect-The-Dot%20靶机渗透总结/IMG-20260826094054867.png)

使用工具解密后，得到一串描述性的提示。

![m3](../vulnhubScreenShot/Vulnhub%20Connect-The-Dot%20靶机渗透总结/IMG-20260826094200851.png)
解码后的结果

```text
HEY NORRIS, YOU'VE MADE THIS FAR. FAR FAR FROM HEAVEN WANNA SEE HELL NOW? HAHA YOU SURELY MISSED ME, DIDN'T YOU? OH DAMN MY BATTERY IS ABOUT TO DIE AND I AM UNABLE TO FIND MY CHARGER SO QUICKLY LEAVING A HINT IN HERE BEFORE THIS SYSTEM SHUTS DOWN AUTOMATICALLY. I AM SAVING THE GATEWAY TO MY DUNGEON IN A 'SECRETFILE' WHICH IS PUBLICLY ACCESSIBLE.
```

morris 提示我们他把一个 secretfile 文件放在一个公开能访问的地方。我们现阶段目标需要获取这个文件。

## norris ssh 获得 secretfile 文件

我们登录 norris ssh 账号。

![ssh1](../vulnhubScreenShot/Vulnhub%20Connect-The-Dot%20靶机渗透总结/IMG-20260826095135646.png)

我们发现 secretfile 文件在 web 目录下面。但是通过搜索，这里的 .secretfile.swp 文件更具有价值只是在 norris 环境下，没法读取。但我能通过网页下载这个文件。

下载到本地，我们提取里面的关键字符串，发现了 morris 用户的凭据。

![ssh2](../vulnhubScreenShot/Vulnhub%20Connect-The-Dot%20靶机渗透总结/IMG-20260826095432924.png)

morris 的登录凭据是 blehguessme090 。

# 权限提升

获得立足点后，我们开始权限枚举。首先通过 find 命令查找所以具有 suid 权限的文件。

```shell
find / -perm -u=s -type f 2>/dev/null
```

我们发现这里的 polkit-agent-helper-1 文件具有 suid 权限。通过查看目标 polkit 的手册，我们得知这个文件是用于给高权限与低权限程序沟通的中介人。我们平常使用 kali 通过命令 systemctl 启动某个服务时，就会弹出一个 dialog。要求我们输入 root 密码。这个 对话框就是 helper 这个软件干的事。说明目标系统上还运行这另一套安全机制 PolicyKit 。

![priv1](vulnhubScreenShot/Vulnhub%20Connect-The-Dot%20靶机渗透总结/IMG-20260827205219903.png)

回顾目前情况，我们手上有两个用户的凭据，但是都没有执行 sudo 的权限。感觉目标对于 sudo 权限的操作非常严格，那么会不会在 polkit 的配置上有疏漏？

我们通过查看本地的手册，发现 polkit 配置是通过读取这个 /etc/polkit-1/localauthority.conf.d 文件夹下面的信息来决定的。并且序号大的文件会覆盖序号小的文件。

![priv2](../vulnhubScreenShot/Vulnhub%20Connect-The-Dot%20靶机渗透总结/IMG-20260827210332990.png)

我们使用命令查看这个文件下的文件内容。

```shell
ls -liah /etc/polkit-1/localauthority.conf.d/
cd /etc/polkit-1/localauthority.conf.d/
cat 51-debian-sudo.conf
```

![priv3](../vulnhubScreenShot/Vulnhub%20Connect-The-Dot%20靶机渗透总结/IMG-20260827210731343.png)

我们发现当前 polkit 指定的管理员组是 sudo 组。我们手上只有 norris 用户具有 sudo 组权限。也就是如果我们使用 norris 执行命令，由于我们是管理员组的原因，会通过 polkit 的判断机制，然后，我们会获得 polkit 的权限认可，进而以高权限执行我们的命令。

我们选择使用 systemd-run 命令来提权。 我们参考了这篇文章[systemd-run](https://github.com/hackerhouse-opensource/exploits/blob/master/systemd-run-tty.txt)
我们输入命令

```shell
systemd-run --shell
```


![priv4](../vulnhubScreenShot/Vulnhub%20Connect-The-Dot%20靶机渗透总结/IMG-20260827211516196.png)

成功提权。

# 总结

这台靶机添加了 jsFuck ，morse code 等新奇的解密关卡。借助这台靶机，我们对 polkit 的机制有所了解。


# 补充

## Capabilities 缺陷利用

Capabilities 简称 cap。就是把 root 用户的能力细分为各个细颗粒度的功能模块。拥有 cap 的可执行文件。普通用户执行后，可以拿到小部分只有 root 才能执行的功能。
通过搜索目标主机，我们发现了 tar 工具具有读取和搜索的功能。CAP_DAC_READ_SEARCH 能够进行读和搜索。我们可以通过 tar 工具将 root 文件下的内容搜索出来。

![cap1](../vulnhubScreenShot/Vulnhub%20Connect-The-Dot%20靶机渗透总结/IMG-20260828105050406.png)

