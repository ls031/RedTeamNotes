
###### DNS枚举
DNS层级系统是一个巨大的分布型数据库，是实现攻击面扩展的高价值目标
对应DNS记录的信息有：
- **NS**: Nameserver records contain the name of the authoritative servers hosting the DNS records for a domain.
- **A**: Also known as a host record, the "_a record_" contains the IPv4 address of a hostname (such as www.megacorpone.com).
- **AAAA**: Also known as a quad A host record, the "_aaaa record_" contains the IPv6 address of a hostname (such as www.megacorpone.com).
- **MX**: 记录存放负责处理该域名邮件收发的服务器域名，一个域名可以配置多条 MX 记录。
- **PTR**: 应用于反向解析区域，可通过 IP 地址反查对应的域名信息。
- **CNAME**: 用于为其他主机记录创建别名。
- **TXT**: 可存放任意自定义文本数据，用途多样，常见于**域名所有权验证**。

原子化工具列举：
kali
```bash
 host -t mx megacorpone.com
 //使用-t参数指定要枚举的dns record
 ┌──(kali㉿kali)-[~]
└─$ host -t mx megacorpone.com
megacorpone.com mail is handled by 10 fb.mail.gandi.net.
megacorpone.com mail is handled by 60 mail2.megacorpone.com.
megacorpone.com mail is handled by 50 mail.megacorpone.com.
megacorpone.com mail is handled by 20 spool.mail.gandi.net.
//数字越小表示越优先向该服务器寄往邮件
```

###### Bash-One-liner
_1_ 反向解析域名
``` 反向解析域名
┌──(kali㉿kali)-[~]
└─$ cat list.txt                     
ftp
www
mail
owa
proxy
router
                                                                                
┌──(kali㉿kali)-[~]
└─$ for ip in $(seq 200 254); do host 51.222.169.$ip; done | grep -v "not found"
200.169.222.51.in-addr.arpa domain name pointer ip200.ip-51-222-169.net.
201.169.222.51.in-addr.arpa domain name pointer smtp-east19.getmailinhub.com.
202.169.222.51.in-addr.arpa domain name pointer smtp-east20.getmailinhub.com.
203.169.222.51.in-addr.arpa domain name pointer smtp-east21.getmailinhub.com.
204.169.222.51.in-addr.arpa domain name pointer smtp-east22.getmailinhub.com.
205.169.222.51.in-addr.arpa domain name pointer smtp-east23.getmailinhub.com.
206.169.222.51.in-addr.arpa domain name pointer smtp-east24.getmailinhub.com.
207.169.222.51.in-addr.arpa domain name pointer ip207.ip-51-222-169.net.
224.169.222.51.in-addr.arpa domain name pointer apps.issl.ng.
225.169.222.51.in-addr.arpa domain name pointer zentyal.issl.ng.
226.169.222.51.in-addr.arpa domain name pointer crvs.issl.ng.
227.169.222.51.in-addr.arpa domain name pointer gemsoutdoor.com.
228.169.222.51.in-addr.arpa domain name pointer postoffice.issl.ng.
229.169.222.51.in-addr.arpa domain name pointer perennial.integrabanking.ng.
230.169.222.51.in-addr.arpa domain name pointer test.boardseats.io.
231.169.222.51.in-addr.arpa domain name pointer ip231.ip-51-222-169.net.
232.169.222.51.in-addr.arpa domain name pointer ip232.ip-51-222-169.net.
233.169.222.51.in-addr.arpa domain name pointer ip233.ip-51-222-169.net.
234.169.222.51.in-addr.arpa domain name pointer ip234.ip-51-222-169.net.
235.169.222.51.in-addr.arpa domain name pointer ip235.ip-51-222-169.net.
236.169.222.51.in-addr.arpa domain name pointer ip236.ip-51-222-169.net.
237.169.222.51.in-addr.arpa domain name pointer ip237.ip-51-222-169.net.
238.169.222.51.in-addr.arpa domain name pointer ip238.ip-51-222-169.net.
239.169.222.51.in-addr.arpa domain name pointer ip239.ip-51-222-169.net.
240.169.222.51.in-addr.arpa domain name pointer ip240.ip-51-222-169.net.
241.169.222.51.in-addr.arpa domain name pointer smtp-east25.getmailinhub.com.
242.169.222.51.in-addr.arpa domain name pointer smtp-east26.getmailinhub.com.
243.169.222.51.in-addr.arpa domain name pointer smtp-east27.getmailinhub.com.
244.169.222.51.in-addr.arpa domain name pointer smtp-east28.getmailinhub.com.
245.169.222.51.in-addr.arpa domain name pointer smtp-east29.getmailinhub.com.
246.169.222.51.in-addr.arpa domain name pointer smtp-east30.getmailinhub.com.
247.169.222.51.in-addr.arpa domain name pointer ip247.ip-51-222-169.net.
248.169.222.51.in-addr.arpa domain name pointer mail.khronostelecom.mx.
249.169.222.51.in-addr.arpa domain name pointer ip249.ip-51-222-169.net.
250.169.222.51.in-addr.arpa domain name pointer ip250.ip-51-222-169.net.
251.169.222.51.in-addr.arpa domain name pointer ip251.ip-51-222-169.net.
252.169.222.51.in-addr.arpa domain name pointer ip252.ip-51-222-169.net.
253.169.222.51.in-addr.arpa domain name pointer ip253.ip-51-222-169.net.
254.169.222.51.in-addr.arpa domain name pointer ip254.ip-51-222-169.net.

```
_2_ 解析ip地址
```
┌──(kali㉿kali)-[~]
└─$ for ip in $(cat list.txt); do host $ip.megacorpone.com;done | grep -v "not found"
ftp.megacorpone.com has address 198.18.0.60
www.megacorpone.com has address 198.18.5.232
mail.megacorpone.com has address 198.18.0.61
owa.megacorpone.com has address 198.18.0.62
proxy.megacorpone.com has address 198.18.0.63
router.megacorpone.com has address 198.18.0.64
                                                                                
┌──(kali㉿kali)-[~]
└─$ for ip in $(cat list.txt); do host $ip.megacorpone.com;done                      
ftp.megacorpone.com has address 198.18.0.60
Host ftp.megacorpone.com not found: 3(NXDOMAIN)
www.megacorpone.com has address 198.18.5.232
mail.megacorpone.com has address 198.18.0.61
owa.megacorpone.com has address 198.18.0.62
Host owa.megacorpone.com not found: 3(NXDOMAIN)
proxy.megacorpone.com has address 198.18.0.63
Host proxy.megacorpone.com not found: 3(NXDOMAIN)
router.megacorpone.com has address 198.18.0.64

```

###### DNS枚举工具
```
//使用标准枚举方式
┌──(kali㉿kali)-[~]
└─$ dnsrecon -d megacorpone.com -t std              
2026-06-04T08:18:28.940422-0400 INFO Starting enumeration for domain: megacorpone.com
2026-06-04T08:18:28.941813-0400 INFO std: Performing General Enumeration against: megacorpone.com...
2026-06-04T08:18:29.389453-0400 ERROR No answer for DNSSEC query for megacorpone.com
2026-06-04T08:18:29.787083-0400 INFO     SOA ns1.megacorpone.com 198.18.0.12
2026-06-04T08:18:30.229897-0400 INFO     NS ns3.megacorpone.com 198.18.0.13
2026-06-04T08:18:30.296205-0400 ERROR    Recursion enabled on NS Server 198.18.0.13
2026-06-04T08:18:30.320053-0400 INFO     NS ns1.megacorpone.com 198.18.0.12
2026-06-04T08:18:30.324892-0400 ERROR    Recursion enabled on NS Server 198.18.0.12
2026-06-04T08:18:30.504839-0400 INFO     NS ns2.megacorpone.com 198.18.0.14
2026-06-04T08:18:30.508758-0400 ERROR    Recursion enabled on NS Server 198.18.0.14
2026-06-04T08:18:31.264871-0400 INFO     MX mail.megacorpone.com 198.18.0.15
2026-06-04T08:18:31.265237-0400 INFO     MX spool.mail.gandi.net 198.18.0.16
2026-06-04T08:18:31.265528-0400 INFO     MX fb.mail.gandi.net 198.18.0.17
2026-06-04T08:18:31.265975-0400 INFO     MX mail2.megacorpone.com 198.18.0.18
2026-06-04T08:18:31.315993-0400 INFO     A megacorpone.com 198.18.0.19
2026-06-04T08:18:32.179801-0400 INFO     TXT megacorpone.com Try Harder
2026-06-04T08:18:32.180312-0400 INFO     TXT megacorpone.com google-site-verification=U7B_b0HNeBtY4qYGQZNsEYXfCJ32hMNV3GtC0wWq5pA
2026-06-04T08:18:33.204478-0400 INFO Enumerating SRV Records
2026-06-04T08:18:36.886499-0400 ERROR No SRV Records Found for megacorpone.com
2026-06-04T08:18:36.887534-0400 INFO Completed enumeration for domain: megacorpone.com
```
```
//使用字典进行暴力枚举
┌──(kali㉿kali)-[~]
└─$ dnsrecon -d megacorpone.com -D ~/list.txt -t brt
2026-06-04T08:20:02.199407-0400 INFO Using the dictionary file: /home/kali/list.txt (provided by user)
2026-06-04T08:20:02.199594-0400 INFO Starting enumeration for domain: megacorpone.com
2026-06-04T08:20:02.199840-0400 INFO brt: Performing host and subdomain brute force against megacorpone.com...
2026-06-04T08:20:02.295030-0400 INFO Do you wish to continue? [Y/n]
Y
2026-06-04T08:20:03.919356-0400 INFO     A ftp.megacorpone.com 198.18.0.21
2026-06-04T08:20:03.919886-0400 INFO     A www.megacorpone.com 198.18.5.232
2026-06-04T08:20:03.920377-0400 INFO     A mail.megacorpone.com 198.18.0.15
2026-06-04T08:20:03.922063-0400 INFO     A router.megacorpone.com 198.18.0.64
2026-06-04T08:20:03.923233-0400 INFO     A proxy.megacorpone.com 198.18.0.22
2026-06-04T08:20:03.925344-0400 INFO     A owa.megacorpone.com 198.18.0.23
2026-06-04T08:20:03.926541-0400 INFO 6 Records Found
2026-06-04T08:20:03.926767-0400 INFO Completed enumeration for domain: megacorpone.com

//或者使用dnsenum工具
dnsenum megacorpone.com
```
_DNS枚举常用字典：_ **/usr/share/seclists**
Windows枚举工具
_LOLBAS:_  **Living Off The Land Binaries, Scripts and Libraries,利用系统的二进制库和文件，实现不上传恶意文件，不落地病毒，使用白程序干黑活。**
基于live off the land，可以使用nslookup工具进行枚举
```
C:\Users\retro>nslookup mail.megacorptwo.com
服务器:  UnKnown
Address:  fdfe:dcba:9876::2

名称:    mail.megacorptwo.com
Address:  198.18.6.3

//和linux的host命令一样
可以指定查询的类型

C:\Users\student>nslookup -type=TXT info.megacorptwo.com 192.168.50.151
Server:  UnKnown
Address:  192.168.50.151

info.megacorptwo.com    text =

        "greetings from the TXT record body"
```

