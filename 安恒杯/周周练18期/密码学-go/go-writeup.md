参考链接：http://skysec.top/2018/09/24/2018%E5%AE%89%E6%81%92%E6%9D%AF-9%E6%9C%88%E6%9C%88%E8%B5%9BWriteup/#Crypto2



# 题目链接

题目名称：

Go（提交你找到的字符串的md5值） 

题目地址：

https://xpro-adl.91ctf.com/userdownload?filename=5ba3589b6fd54.zip&type=attach&feature=2018-09

# 解题思路

解压文件

可以得到一个文件和一串字符串。

文件的内容如下：

```
ilnllliiikkninlekile
```

字符串位于压缩文件的注释中，需要额外注意一下子才看得到

```
546865206c656e677468206f66207468697320706c61696e746578743a203130
```

这串字符串看上去应该是一段十六进制的字符串，因此我们需要首先将这段十六进制字符串进行解密

```
print "546865206c656e677468206f66207468697320706c61696e746578743a203130".decode("hex")
```

解密以后，可以得到一个提示

```
The length of this plaintext: 10
```

这个提示是说明文长度是10，那么我们现在拿到的密文长度是

```
print len("ilnllliiikkninlekile")
```

通过python计算可以知道是20.

那么现在我们知道当前密文的长度刚好是明文的长度的2倍。

根据这个推断，再加上脑洞。我们可以知道这次的加密算法采用的是`多文字加密法`

多文字加密法就是通过一个5x5的矩阵表，来将明文替换成密文对，需要一个密钥，比如，这里的密钥就是codes。

![1538156126297](F:\学习资料\hacker\学习笔记\ctf\writeup\安恒杯\周周练18期\密码学-go\1538156126297.png)

比如明文a的密文对就是cc。

那么我们现在需要做的就是根据当前的密文来推断出密钥是多少。

现在的信息是密文长度是明文的2倍，密文中只有5个字母，那么可以猜测它的密钥应该就是这5个字母。只是排序规则不确定，也就是这个5个字母用于组成5x5的矩阵表，但是这5个字母到底是按照什么顺序组成的无法确定。那么只能爆破一波。可以利用脚本

```
import itertools

key = []
cipher = "ilnllliiikkninlekile"

for i in itertools.permutations('ilnke', 5):
    key.append(''.join(i))

for now_key in key:
    solve_c = ""
    res = ""
    for now_c in cipher:
        solve_c += str(now_key.index(now_c))
    for i in range(0,len(solve_c),2):
        now_ascii = int(solve_c[i])*5+int(solve_c[i+1])+97
        if now_ascii>ord('i'):
            now_ascii+=1
        res += chr(now_ascii)
    if "flag" in res:
        print now_key,res
```

通过上述的脚本，可以得到最终的密钥和明文

```
linke flagishere
```

接下来，再将明文进行md5计算，即可得到flag

```
from hashlib import md5

plain = "flagishere"
md5 = md5(plain.encode('utf8'))
md52 = md5.hexdigest()
print md52
```

最终得到flag的值

```
eedda7bea3964bfb288ca6004a973c2a
```





