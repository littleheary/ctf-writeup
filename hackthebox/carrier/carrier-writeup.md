# 介绍
carrier的链接

https://www.hackthebox.eu/home/machines/profile/155

IP地址：10.10.10.105

# 端口扫描
```
nmap -sT -sU -T4 10.10.10.105 -v
```
发现该主机开放的端口如下：
```
TCP
21 ftp 过滤状态
22 ssh
80 web
UDP
161 snmp
67 dhcp 过滤状态
32774 rpc12 过滤状态
```
# 渗透测试
通过浏览器打开`http://10.10.10.105/`，发现是一个登陆页面。

![1549870730186](F:\学习资料\github\ctf-writeup\hackthebox\carrier\assets\1549870730186.png)

进行目录遍历，发现存在如下目录信息

```
python3 dirsearch.py -u http://10.10.10.105 -e php
```

```
[15:26:42] 301 -  310B  - /css  ->  http://10.10.10.105/css/
[15:26:43] 302 -    0B  - /dashboard.php  ->  /index.php
[15:26:48] 301 -  312B  - /debug  ->  http://10.10.10.105/debug/
[15:26:52] 200 -    1KB - /doc/
[15:26:53] 301 -  310B  - /doc  ->  http://10.10.10.105/doc/
[15:27:02] 200 -   83KB - /debug/
[15:27:17] 301 -  312B  - /fonts  ->  http://10.10.10.105/fonts/
[15:27:33] 301 -  310B  - /img  ->  http://10.10.10.105/img/
[15:27:37] 200 -    1KB - /index.php/login/
[15:27:38] 200 -    1KB - /index.php
[15:27:45] 301 -  309B  - /js  ->  http://10.10.10.105/js/
[15:29:05] 403 -  300B  - /server-status
[15:29:06] 403 -  301B  - /server-status/
[15:29:46] 301 -  312B  - /tools  ->  http://10.10.10.105/tools/
```

通过分析上述内容，可以看出，目前可访问的目录有

```
/doc/
/debug/
/img/
/js/
/tools/
```

尝试访问`/doc/`，发现有两个文件。

![1549870897285](F:\学习资料\github\ctf-writeup\hackthebox\carrier\assets\1549870897285.png)

其中的图片是一个拓扑图，pdf是一个错误代码的列表

![1549870997476](F:\学习资料\github\ctf-writeup\hackthebox\carrier\assets\1549870997476.png)

观察到其中`45009`，可以看到提示说默认密码是序列号。

通过之前的端口扫描，可以知道这台主机同时开放着snmp服务，可以尝试读取snmp看是否允许被访问到一些信息。

```
snmpwalk -v 1 -c public 10.10.10.105
```

成功读取到一些信息。

```
iso.3.6.1.2.1.47.1.1.1.1.11 = STRING: "SN#NET_45JDX23"
End of MIB
```

应该就是序列号，那么我们返回登陆页面，尝试登陆。

用户名/密码：`admin/NET_45JDX23`

成功登陆进入。

![1549871250603](F:\学习资料\github\ctf-writeup\hackthebox\carrier\assets\1549871250603.png)

查看`Tickets`页面，发现一些信息提示。

```
10.120.15,10.120.16,10.120.17/24's, one of their VIP is having issues connecting by FTP to an important server in the 10.120.15.0/24 network
```

提示内容是说10.120.15.0/24网段存在一个重要的ftp服务器。

继续查看`Diagnostics`页面，点击“verify status”，发现会回显一些命令格式的结果。猜测有可能有命令注入的漏洞。

抓包获取数据包，发现有可控参数`check`，原有参数是`cXVhZ2dh`

经过base64解码，得出是`quagga`，这与输出中的部分内容可以匹配。

构造一段命令尝试是否存在命令注入`quagga;netstat -ano`，将其转换为base64，`cXVhZ2dhO25ldHN0YXQgLWFubw==`

提交结果，发现确实存在命令注入，成功获得输出。

![1549873194231](F:\学习资料\github\ctf-writeup\hackthebox\carrier\assets\1549873194231.png)

尝试构造命令，搜索`user.txt`文件，发现在`/root/user.txt`

继续构造命令，查看该文件内容，`cat /root/user.txt`，base64转换。

成功读取到该文件的内容，获取到了第一步的flag。

继续尝试搜索`root.txt`，发现没有，猜测应该在这台机子的内网里。应该是需要以这台主机为跳板进行内网渗透了。

到此为止，时间原因，未继续做下一步的渗透测试工作。

