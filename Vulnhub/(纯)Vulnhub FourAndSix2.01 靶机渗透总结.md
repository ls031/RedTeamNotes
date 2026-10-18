
## 靶机描述

_这是一台标榜难度为简单的开源靶机。我们的目标是获得 root 权限，并且提升为 root 用户。_



## 信息收集

1. 端口扫描

```text
# Nmap 7.99 scan initiated Sat Oct 17 14:59:39 2026 as: /usr/lib/nmap/nmap --min-rate 10000 -sT -oA nmapscan/TCPS 192.168.1.7
Nmap scan report for 192.168.1.7
Host is up (0.0025s latency).
Not shown: 887 filtered tcp ports (no-response), 109 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
111/tcp  open  rpcbind
987/tcp  open  unknown
2049/tcp open  nfs
MAC Address: 00:0C:29:F1:99:FB (VMware)

# Nmap done at Sat Oct 17 14:59:40 2026 -- 1 IP address (1 host up) scanned in 0.89 seconds
```

我们看到目标放行了许多端口，其中 111，2049 端口应当排在第一优先级。因为这类端口对应着 nfs 服务。很有可能我们可以找到一下泄露的文件信息。

2. 详细脚本扫描

```text
# Nmap 7.99 scan initiated Sat Oct 17 15:01:23 2026 as: /usr/lib/nmap/nmap -sT -sV -O -sC -p22,111,987,2049 -oA nmapscan/details 192.168.1.7
Nmap scan report for 192.168.1.7
Host is up (0.00068s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.9 (protocol 2.0)
| ssh-hostkey: 
|   2048 ef:3b:2e:cf:40:19:9e:bb:23:1e:aa:24:a1:09:4e:d1 (RSA)
|   256 c8:5c:8b:0b:e1:64:0c:75:c3:63:d7:b3:80:c9:2f:d2 (ECDSA)
|_  256 61:bc:45:9a:ba:a5:47:20:60:13:25:19:b0:47:cb:ad (ED25519)
111/tcp  open  rpcbind 2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100003  2,3         2049/tcp   nfs
|   100003  2,3         2049/udp   nfs
|   100005  1,3          625/udp   mountd
|_  100005  1,3          987/tcp   mountd
987/tcp  open  mountd  1-3 (RPC #100005)
2049/tcp open  nfs     2-3 (RPC #100003)
MAC Address: 00:0C:29:F1:99:FB (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: OpenBSD 6.X
OS CPE: cpe:/o:openbsd:openbsd:6
OS details: OpenBSD 6.0 - 6.4
Network Distance: 1 hop

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Oct 17 15:01:37 2026 -- 1 IP address (1 host up) scanned in 14.03 seconds
```

我们使用以下命令枚举 nfs 服务的开放文件信息。

```shell
showmount -e 192.168.1.7
```

确认 nfs 开放的文件目录是 /home/user/storage，我们使用 mount 命令挂载在我们本地 KALI 上面。

```shell
mkdir /mnt/nfs
sudo mkdir /mnt/nfs                                                                sudo mount -t nfs 192.168.1.7:/home/user/storage /mnt/nfs
```

我们成功发现了一个 backup.zip 的文件。

![p1](../vulnhubScreenShot/(纯)Vulnhub%20FourAndSix2.01%20靶机渗透总结/IMG-20261018210002356.png)

## 7z 压缩包密码破解

使用 7z 的 x 选项解压 backup.7z 压缩包。我们发现压缩包找我们要密码。使用 Google 以 “how to crack 7z file” 为关键词。我找到了合适的文章，使用 john-the-ripper 工具破解。

![p2](../vulnhubScreenShot/(纯)Vulnhub%20FourAndSix2.01%20靶机渗透总结/IMG-20261018210404462.png)

![p2.1](../vulnhubScreenShot/(纯)Vulnhub%20FourAndSix2.01%20靶机渗透总结/IMG-20261018210901052.png)

按照博客的步骤，我们得到了密码 chacolate。

![p3](../vulnhubScreenShot/(纯)Vulnhub%20FourAndSix2.01%20靶机渗透总结/IMG-20261018210503486.png)

解压后，发现了一对公私钥，和一些图片。这对公私钥我们非常感兴趣。我们可以借助它获得立足点。通过公钥，我们知道了ssh 的用户名是 user。

![p4](../vulnhubScreenShot/(纯)Vulnhub%20FourAndSix2.01%20靶机渗透总结/IMG-20261018210731807.png)



## ssh 私钥 passphrase 破解

我们指定私钥，发现找我们要 passphrase。

![p5](../vulnhubScreenShot/(纯)Vulnhub%20FourAndSix2.01%20靶机渗透总结/IMG-20261018211139770.png)

我们拿到私钥，如果 passphrase 不复杂的话，我们也是可以暴力枚举出来的。和上面的 7z2john 方式差不多。我们可以说使用 ssh2john 来尝试破解 passphrase。

![p6](../vulnhubScreenShot/(纯)Vulnhub%20FourAndSix2.01%20靶机渗透总结/IMG-20261018211534807.png)

我们通过 passphrase 成功登录 ssh 会话。

![p7](../vulnhubScreenShot/(纯)Vulnhub%20FourAndSix2.01%20靶机渗透总结/IMG-20261018211702831.png)


## 权限提升

我们发现当前系统的内核版本低，可以采用内核提权的方式获得 root 权限。

![p8](../vulnhubScreenShot/(纯)Vulnhub%20FourAndSix2.01%20靶机渗透总结/IMG-20261018211938655.png)

![p9](../vulnhubScreenShot/(纯)Vulnhub%20FourAndSix2.01%20靶机渗透总结/IMG-20261018212045270.png)

这里经过尝试，只有 47780 这个漏洞提权成功。

![p10](../vulnhubScreenShot/(纯)Vulnhub%20FourAndSix2.01%20靶机渗透总结/IMG-20261018212152805.png)

![p11](../vulnhubScreenShot/(纯)Vulnhub%20FourAndSix2.01%20靶机渗透总结/IMG-20261018212223116.png)



## 总结

我们通过信息收集，发现目标运行着 22，111，2049 端口服务。通过 111，2049 端口我们发现泄露的敏感信息 backup.zip 文件。通过 hash 破解，我们得到了压缩包的密码。成功获得里面 公私钥对。通过破解 passphrase ，我们获得了 user 的ssh 的 passphrase。获得了立足点。通过内核提权，我们获得了 root 权限。

