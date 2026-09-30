
### 靶场描述

_这是一台标榜难度为简单的 vulnhub 开源靶机。作者说了这台靶机十分的基础。_

![des](../vulnhubScreenShot/(纯)%20NyxVM%20靶机渗透总结/IMG-20260930214457354.png)


### 信息收集

1. 端口扫描

我们先使用 TCP 协议进行扫描。我们发现目标开放了 22，80 端口。
```text
# Nmap 7.99 scan initiated Sun Sep 27 20:54:06 2026 as: /usr/lib/nmap/nmap --min-rate 10000 -sT -p- -oA nmapscan/TCPS 10.10.10.178
Nmap scan report for 10.10.10.178
Host is up (0.00061s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 00:0C:29:A3:D0:19 (VMware)

# Nmap done at Sun Sep 27 20:54:09 2026 -- 1 IP address (1 host up) scanned in 3.43 seconds

```

2. 详细信息扫描
```text
# Nmap 7.99 scan initiated Sun Sep 27 20:56:25 2026 as: /usr/lib/nmap/nmap -sT -sV -O -p22,80 -oA nmapscan/details 10.10.10.178
Nmap scan report for 10.10.10.178
Host is up (0.00078s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
MAC Address: 00:0C:29:A3:D0:19 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Sep 27 20:56:33 2026 -- 1 IP address (1 host up) scanned in 8.18 seconds

```

3. 漏洞脚本扫描

漏洞脚本扫描告诉我们有个敏感文件，能够任意重置管理员的密码。

```text
# Nmap 7.99 scan initiated Sun Sep 27 20:57:18 2026 as: /usr/lib/nmap/nmap --script=vuln -sT -oA nmapscan/vulns 10.10.10.178
Nmap scan report for 10.10.10.178
Host is up (0.00097s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-enum: 
|_  /d41d8cd98f00b204e9800998ecf8427e.php: Seagate BlackArmorNAS 110/220/440 Administrator Password Reset Vulnerability
MAC Address: 00:0C:29:A3:D0:19 (VMware)

# Nmap done at Sun Sep 27 20:57:49 2026 -- 1 IP address (1 host up) scanned in 31.36 seconds

```


### WEB 渗透

#### 敏感文件泄露私钥

这是一个 ssh 的私钥文件。我们可以凭借这个私钥快速获取立足点。这里的 title 上写着 mpampis key。 明摆着这是 mpampis 的私钥。

![p1](../vulnhubScreenShot/(纯)%20NyxVM%20靶机渗透总结/IMG-20260930220757402.png)


#### 密码爆破 兔子洞

在访问 80 端口时，就发现作者提示我们少关注源码，多聚焦于真实场景。通过目录爆破，发现有个 key.php 的页面。

![p2](../vulnhubScreenShot/(纯)%20NyxVM%20靶机渗透总结/IMG-20260930221021486.png)


![p3](../vulnhubScreenShot/(纯)%20NyxVM%20靶机渗透总结/IMG-20260930221150604.png)

我们通过 hydra 暴力破解，成功获取口令 admin，但是当前界面没有给我们下一步的提示。

![p4](../vulnhubScreenShot/(纯)%20NyxVM%20靶机渗透总结/IMG-20260930221252540.png)


### ssh 获取立足点

#### 小插曲权限开放拒绝拒绝链接

我们将私钥文件保存下来直接连接 ssh 时，发现提示我们权限太开放了。我进行权限缩进 400 只能自己读取私钥。

![p5](../vulnhubScreenShot/(纯)%20NyxVM%20靶机渗透总结/IMG-20260930221409635.png)


成功获得立足点

![p6](../vulnhubScreenShot/(纯)%20NyxVM%20靶机渗透总结/IMG-20260930221711333.png)

![p7](../vulnhubScreenShot/(纯)%20NyxVM%20靶机渗透总结/IMG-20260930221841348.png)
### 权限提升

通过登录界面 sudo -l 权限枚举，发现我们可以用 root 权限使用 gcc 工具。
这里构造 priv.c 是错误的思路，我们编译好的 priv 我们没法运行。

![p8](../vulnhubScreenShot/(纯)%20NyxVM%20靶机渗透总结/IMG-20260930221910420.png)

我们通过 Google 搜索到了 gcc 的提权方法。

![p9](../vulnhubScreenShot/(纯)%20NyxVM%20靶机渗透总结/IMG-20260930222051725.png)

![p10](../vulnhubScreenShot/(纯)%20NyxVM%20靶机渗透总结/IMG-20260930222113705.png)


成功验证 root 权限。

![p11](../vulnhubScreenShot/(纯)%20NyxVM%20靶机渗透总结/IMG-20260930222144032.png)

### 总结

这台靶机比较简单，攻击链路非常清晰。

