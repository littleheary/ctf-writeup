参考链接：http://skysec.top/2018/09/24/2018%E5%AE%89%E6%81%92%E6%9D%AF-9%E6%9C%88%E6%9C%88%E8%B5%9BWriteup/

------





# 题目链接

https://xpro-adl.91ctf.com/userdownload?filename=5ba3589b5a9bb.zip&type=attach&feature=2018-09

# 解题思路

下载附件下来以后，解压，发现里面是一个python脚本，名称为encode.py，猜测是一个加密的脚本。

```
#!/usr/bin/env python
# -*- coding:utf-8 -*- 
from Crypto.Cipher import AES
from Crypto import Random

def encrypt(data, password):
    bs = AES.block_size
    pad = lambda s: s + (bs - len(s) % bs) * chr(bs - len(s) % bs)
    iv = "0102030405060708"
    cipher = AES.new(password, AES.MODE_CBC, iv)
    data = cipher.encrypt(pad(data))
    return data
 
def decrypt(data, password):
    unpad = lambda s : s[0:-ord(s[-1])]
    iv = "0102030405060708"
    cipher = AES.new(password, AES.MODE_CBC, iv)
    data  = cipher.decrypt(data)
    return unpad(data)
    
def generate_passwd(key):
    data_halt = "LvR7GrlG0A4WIMBrUwTFoA==".decode("base64")
    rand_int =  int(decrypt(data_halt, key).encode("hex"), 16)
    round = 0x7DC59612
    result = 1    
    a1 = 0
    while a1 < round:
        a2 = 0
        while a2 < round:
            a3 = 0
            while a3 < round:
                result = result * (rand_int % 0xB18E) % 0xB18E
                a3 += 1
            a2 += 1
        a1 += 1
    return encrypt(str(result), key)
    

if __name__ == '__main__':

    key = raw_input("key:")
    
    if len(key) != 32:
        print "check key length!"
        exit()
    passwd = generate_passwd(key.decode("hex"))
    
    flag = raw_input("flag:")
    
    print "output:", encrypt(flag, passwd).encode("base64")
    
            
        
# key = md5(sha1("flag"))
# output = "u6WHK2bnAsvTP/lPagu7c/K3la0mrveKrXryBPF/LKFE2HYgRNLGzr1J1yObUapw"


```

最下方有提示，这个脚本的输入是多少，最后的输出是

```
u6WHK2bnAsvTP/lPagu7c/K3la0mrveKrXryBPF/LKFE2HYgRNLGzr1J1yObUapw
```

那么我们只能想办法来逆推回去咯。

脚本中已经给了加解密的函数，其中还有一个构造密码的函数。

仔细查看构造密码的函数，其中的result的值肯定小于0xB18E,那么我们可以通过爆破来将flag算出。

在原来的脚本中，去掉main下面的内容，输入我们构造的代码

首先把key计算一下

```
from hashlib import sha1
from hashlib import md5

flag = "flag"
sha1_1 = sha1(flag.encode('utf8'))
sha1_2 = sha1_1.hexdigest()
md5_1 = md5(sha1_2.encode('utf8'))
md5_2 = md5_1.hexdigest()
print md5_2
```

可以通过上述计算，得出key为`17abeca4cc4c432a52c2b7f6d24d1888`

然后再写下面的爆破脚本

```
output = "u6WHK2bnAsvTP/lPagu7c/K3la0mrveKrXryBPF/LKFE2HYgRNLGzr1J1yObUapw"
key = md5(sha1("flag"))
for result in range(0xB18E):
    passwd = generate_passwd(key.decode("hex"),result)
    r = decrypt(output.decode("base64"), passwd)
    if 'flag' in r:
        print r
```

完整的代码如下

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*- 
from Crypto.Cipher import AES
from Crypto import Random

def encrypt(data, password):
    bs = AES.block_size
    pad = lambda s: s + (bs - len(s) % bs) * chr(bs - len(s) % bs)
    iv = "0102030405060708"
    cipher = AES.new(password, AES.MODE_CBC, iv)
    data = cipher.encrypt(pad(data))
    return data
 
def decrypt(data, password):
    unpad = lambda s : s[0:-ord(s[-1])]
    iv = "0102030405060708"
    cipher = AES.new(password, AES.MODE_CBC, iv)
    data  = cipher.decrypt(data)
    return unpad(data)
    
def generate_passwd(key,result):
	data_halt = "LvR7GrlG0A4WIMBrUwTFoA==".decode("base64")
	rand_int =  int(decrypt(data_halt, key).encode("hex"), 16)
	round = 0x7DC59612
	result = result * (rand_int % 0xB18E) % 0xB18E
	return encrypt(str(result), key)
    

output = "u6WHK2bnAsvTP/lPagu7c/K3la0mrveKrXryBPF/LKFE2HYgRNLGzr1J1yObUapw"
key = "17abeca4cc4c432a52c2b7f6d24d1888"
for result in range(0xB18E):
    passwd = generate_passwd(key.decode("hex"),result)
    r = decrypt(output.decode("base64"), passwd)
    if 'flag' in r:
        print r
```

最后通过python进行计算，可以得出flag

```
flag{552d3a0e567542d99694c4d61d1a652e}
```

