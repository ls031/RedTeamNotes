
### 任意文件读取漏洞

1. 常用参考的资源库文件
 [Auto_Wordlists](https://github.com/carlospolop/Auto_Wordlists)


2. 在php 环境中，如果文件采用 inclue 来进行文件包含的话，那么没法读取 php文件功能的代码。


3. bash 环境低于 4.3 ，可以考虑使用 shellshock 漏洞






密码破解

```text
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

john --wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-MD5 creds.txt
```


