##### TCP和UDP扫描的原理分析
- _TCP_ 使用三次握手原理，来检查基于TCP协议运行服务端口是否开放。如果开放，则会返回一个ack+syn的packet。否则会返回一个rsk+ack的packet。表示端口关闭。

- _UDP_ 将发送一个空白pacekt，如何目标的运行基于UDP协议的服务端口关闭，那么将会返回一个基于ICMP协议的“ICMP port unreachable”。否则则显示端口开放。由于实际环境下，防火墙过滤不放行ICMP报错packet,UDP扫描并不完全可靠。经过UDP扫描的开放的端口不完全可信。


##### Nmap实现TCP，UDP扫描
###### 端口扫描
```
//默认不指定参数会扫描1000个端口
kali@kali:~$ nmap 192.168.50.149
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-09 05:12 EST
Nmap scan report for 192.168.50.149
Host is up (0.10s latency).
Not shown: 989 closed tcp ports (conn-refused)
PORT     STATE SERVICE
53/tcp   open  domain
88/tcp   open  kerberos-sec
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
389/tcp  open  ldap
445/tcp  open  microsoft-ds
464/tcp  open  kpasswd5
593/tcp  open  http-rpc-epmap
636/tcp  open  ldapssl
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl

Nmap done: 1 IP address (1 host up) scanned in 10.95 seconds
```

半开扫描和udp扫描
```
//半开扫描，收到SYN+ACK时候，就确认端口开放
sudo nmap -sS 192.168.50.149
//使用链接扫描，建立链接确认端口开放
nmap -sT 192.168.50.149
//使用UDP扫描，对于特定端口161，它会放出一个特定的绑定链接的包来确认目标服务端口是否开放
sudo nmap -sU 192.168.50.149
//综合起来
sudo nmap -sU -sS 192.168.50.149
```

##### 主机范围扫描
```
//为了使用grep来配合nmap主机扫描结果，使用-oG参数指定文件格式为更好操控的格式
nmap -v -sn 192.168.50.1-253 -oG ping-sweep.txt
//使用grep搜索结果中的UP开放端口，并且将结果以空格进行分割，选取第二列进行展示。
grep Up ping-sweep.txt | cut -d " " -f 2
//扫描网段中所以开放了80端口的主机
nmap -p 80 192.168.50.1-253 -oG web-sweep.txt
//判断开放端口的结果
grep open web-sweep.txt | cut -d" " -f 2
```
综合主机端口扫描
```
nmap -sT -A --top-ports=20 192.168.50.1-253 -oG top-port-sweep.txt
//-A表示 版本选择，追踪扫描，脚本扫描

//对主机操作系统版本进行强制性枚举
sudo nmap -O 192.168.50.14 --osscan-guess
//当我们确认了底层操作系统版本，使用-A进行服务枚举
nmap -sT -A 192.168.50.14
//如果我们专注于服务版本的扫描则可以使用-sV进行操作
```

###### NSE扫描脚本

```
nmap --script-help http-headers
//查看脚本http-headers的帮助信息
```

###### Windows主机扫描
当当前主机没法使用Nmap时候，或者说在内网环境中时，可以使用Windows系统自带的powershell进行主机端口扫描。
```
PS C:\Users\student> Test-NetConnection -Port 445 192.168.50.151

ComputerName     : 192.168.50.151
RemoteAddress    : 192.168.50.151
RemotePort       : 445
InterfaceAlias   : Ethernet0
SourceAddress    : 192.168.50.152
TcpTestSucceeded : True
```

###### Windows one-liner command
```
PS C:\Users\student> 1..1024 | % {echo ((New-Object Net.Sockets.TcpClient).Connect("192.168.50.151", $_)) "TCP port $_ is open"} 2>$null
TCP port 88 is open
```

