
# 靶机描述
 这是一台标榜难度为简单的靶机。使用 PHP+Apache+MySQL 搭建。作者需要我们通过 web 程序的漏洞来逃逸，然后提权到 root 用户。
![des](../vulnhubScreenShot/Billux_b0x/20260727185822.png)


# 信息收集

## nmap 脚本扫描

1. 主机发现
目标主机的 ip 地址是192.168.2.44 。

2. 端口扫描
```text
# Nmap 7.98 scan initiated Sun Jul 26 05:46:40 2026 as: /usr/lib/nmap/nmap --min-rate=10000 -p- -oA nmapscan/TCPS 192.168.2.44
Nmap scan report for 192.168.2.44
Host is up (0.00082s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 00:0C:29:2C:73:2A (VMware)

# Nmap done at Sun Jul 26 05:46:42 2026 -- 1 IP address (1 host up) scanned in 1.60 seconds
```

发现目标主机开放 22， 80 端口。

3. 详细扫描

```text
# Nmap 7.98 scan initiated Sun Jul 26 06:19:48 2026 as: /usr/lib/nmap/nmap -sT -sV -O -sC -p22,80 -oA nmapscan/details 192.168.2.44
Nmap scan report for 192.168.2.44
Host is up (0.00088s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 fa:cf:a2:52:c4:fa:f5:75:a7:e2:bd:60:83:3e:7b:de (DSA)
|   2048 88:31:0c:78:98:80:ef:33:fa:26:22:ed:d0:9b:ba:f8 (RSA)
|_  256 0e:5e:33:03:50:c9:1e:b3:e7:51:39:a4:4a:10:64:ca (ECDSA)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: --==[[IndiShell Lab]]==--
MAC Address: 00:0C:29:2C:73:2A (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14, Linux 3.8 - 3.16
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Jul 26 06:19:56 2026 -- 1 IP address (1 host up) scanned in 8.54 seconds
```

目标主机可能是一台 Ubuntu 系统的 linux 主机，目前扫描出主机模糊的内核版本为 3.x 。
网站运行在 apache 服务器上。

4. 漏洞脚本扫描

```text
# Nmap 7.98 scan initiated Sun Jul 26 06:20:35 2026 as: /usr/lib/nmap/nmap --script=vuln -p22,80 -oA nmapscan/vulns 192.168.2.44
Nmap scan report for 192.168.2.44
Host is up (0.00022s latency).

PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
| http-internal-ip-disclosure: 
|_  Internal IP Leaked: 127.0.1.1
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum: 
|   /test.php: Test page
|_  /images/: Potentially interesting directory w/ listing on 'apache/2.2.22 (ubuntu)'
MAC Address: 00:0C:29:2C:73:2A (VMware)

# Nmap done at Sun Jul 26 06:21:07 2026 -- 1 IP address (1 host up) scanned in 31.49 seconds

```
 漏洞脚本扫描标注 80 端口暴露了一个文件 test.php 以及存放图片的 /images 文件夹。

## WEB 页面信息收集

1. test.php 任意文件读取漏洞

![file](../vulnhubScreenShot/Billux_b0x/Screenshot_2026-07-27_08_22_39.png)

要求我们提交的请求中 file 参数不为空。网站比较常用的两种方法是 GET 和 POST 方法。经过尝试，网站后端接受 POST 方式提交的 file 参数。

![test.php](../vulnhubScreenShot/Billux_b0x/file_read.png)

通过这个参数，我们可以查看网站后端许多资源文件的源码。

2. IMAGES 目录

images 目录下存放了一些图片文件。暂时不清楚用途。



# WEB 渗透

## WEB 页面资源路径爆破

![dir](../vulnhubScreenShot/Billux_b0x/dir.png)

我们可以通过任意文件读取漏洞查看这些路径文件的源码，从而在 WEB 页面中挖漏洞。

## 根目录界面 SQL 注入

我们访问 80 端口，页面提示我们说，“展示我们的 SQL 注入技巧”。我们结合文件读取漏洞来分析。

![WEB](../vulnhubScreenShot/Billux_b0x/Screenshot_2026-07-27_08_40_04.png)

![index](../vulnhubScreenShot/Billux_b0x/index.png)

代码揭示了，后端会把我们输入的账号密码进行检查过滤。把输入中的单引号给替换成空。源码中的 $run 变量 展示了一个字符型注入。我们需要通过闭合单引号来逃逸。从而拼接我们自己的语句。笔者这边钻牛角尖了，最后是通过红笔师傅视频答出来的。其实这类问题应该通过字符型注入的原理出发。常见的思路是通过单引号闭合变量，然后拼接一个逻辑，使得语句的执行结果为正确。但是目标禁止输入 _'_ 。就没法通过输入的方式来闭合。但是，这边提供了两个变量给我们进行注入。我们可以借助后面一个变量的前面的单引号进行闭合，然后凭借我们的注入逻辑，实现登录绕过。

```sql
Password: \

Username: or 1=1 #\
```

反斜杠可以把单引号进行转义。这里反斜杠使得前面的变量 $pass 后的单引号失去闭合作用。从而实现逃逸。

```sql
闭合后的 sql 语句

select * from auth where pass=  '\' and uname='  or 1=1 #\'

 # 注意这里的pass 变量匹配的是 \' and uname= , 然后和 or 逻辑链接，因为 1=1 恒为正，所以这个语句执行结果变成把 auth 这张表的内容全部查出来。
```

## panel.php 文件上传

在panel.php界面中，点击复选框中的 add 选项。出现上传功能界面。

![[kali-linux-2026.1-vmware-amd64-2026-07-27-22-39-04.png]]

查看源码
![[file_upload.png]]
这里查阅 php 手册后发现pathinfo函数会根据文件名和指定flags 类型返回相应的字符串，这里PATHINFO_EXITENSION 是文件后缀的意思。所以这个函数会返回上传文件的后缀名。下面通过finfo 类的接口调用 file 函数返回文件内容类型。
总的来说， 后端检查文件上传不仅会看后缀字符是否匹配，还会看提交的内容是不是图片。
二次渲染，shell成鬼shell问题