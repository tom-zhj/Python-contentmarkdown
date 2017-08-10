# Python中base64加密解密方法详解及版本间差异

今天来看一下base64加密函数的使用，以及Python2与Python3中的不同。

### 一、base64

Base64是一种基于64个可打印字符来表示二进制数据的表示方法。由于2的6次方等于64，所以每6个比特为一个单元，对应某个可打印字符。三个字节有24个比特
，对应于4个Base64单元，即3个字节需要用4个可打印字符来表示。它可用来作为电子邮件的传输编码。在Base64中的可打印字符包括字母A-Z、a-z、数字
0-9 ，这样共有62个字符，此外两个可打印符号在不同的系统中而不同。编码后的数据比原始数据略长，为原来的4/3。

Base64常用于在通常处理文本数据的场合，表示、传输、存储一些二进制数据（或不可打印的字符串）。包括MIME的email，email via MIME,
在XML中存储复杂数据.

在邮件中的用途：

在MIME格式的电子邮件中，base64可以用来将binary的字节序列数据编码成ASCII字符序列构成的文本。使用时，在传输编码方式中指定base64。使
用的字符包括大小写字母各26个，加上10个数字，和加号“+”，斜杠“/”，一共64个字符，等号“=”用来作为后缀用途。

在URL中的用途：

标准的Base64并不适合直接放在URL里传输，因为URL编码器会把标准Base64中的“/”和“+”字符变为形如“%XX”的形式，而这些“%”号在存入数据
库时还需要再进行转换，因为ANSI SQL中已将“%”号用作通配符。

为解决此问题，可采用一种用于URL的改进Base64编码，它不在末尾填充'='号，并将标准Base64中的“+”和“/”分别改成了“*”和“-”，这样就免去
了在URL编解码和数据库存储时所要作的转换，避免了编码信息长度在此过程中的增加，并统一了数据库、表单等处对象标识符的格式。

另有一种用于正则表达式的改进Base64变种，它将“+”和“/”改成了“!”和“-”，因为“+”，“*”在正则表达式中都可能具有特殊含义。

### 二、python中使用

  

Python2 和 Python3 在base64处理上是不同的，Python3下传入的参数不能是Unicode字符串，需要进行转换

#### Python2：

    
    
    Python 2.7.10 (default, Oct 23 2015, 19:19:21) 
    [GCC 4.2.1 Compatible Apple LLVM 7.0.0 (clang-700.0.59.5)] on darwin
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import base64
    >>> str = 'pythontab.com'
    >>> base64.b64encode(str)
    'cHl0aG9udGFiLmNvbQ=='
    >>> base64.b64decode('cHl0aG9udGFiLmNvbQ==')
    'pythontab.com'
    >>>

  

#### Python3：

    
    
    Python 3.5.2 (default, Aug 24 2016, 16:48:29) 
    [GCC 4.2.1 Compatible Apple LLVM 7.3.0 (clang-703.0.31)] on darwin
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import base64
    >>> str = 'pythontab.com'
    >>> bytesStr = str.encode(encoding='utf-8')
    >>> bytesStr.decode()
    'pythontab.com'
    >>> b64str = base64.b64encode(bytesStr)
    >>> b64str
    b'cHl0aG9udGFiLmNvbQ=='
    >>> base64.b64decode(b64str)
    b'pythontab.com'
    >>>

  

**注意：**

首先要搞清楚，字符串在Python内部的表示是unicode编码.

因此，在做编码转换时，通常需要以unicode作为中间编码，即先将其他编码的字符串解码（decode）成unicode，再从unicode编码（encode
）成另一种编码。

decode的作用是将其他编码的字符串转换成unicode编码，

如str1.decode('gb2312')，表示将gb2312编码的字符串转换成unicode编码。

encode的作用是将unicode编码转换成其他编码的字符串，

如str2.encode('gb2312')，表示将unicode编码的字符串转换成gb2312编码。

  

### 三、其他的方法

base64.b64encode(s[, altchars])

base64.b64decode(s[, altchars])

altchars为可选的参数，用来替换+和/的一个两个长度的字符串。

base64.urlsafe_b64encode(s)

base64.urlsafe_b64decode(s)

此方法中用-代替了+，用_代替了/ ，这样可以保证编码后的字符串放在url里可以正常访问

base64.b32encode(s)

base64.b32decode(s[, casefold[, map01]])

base64.b16encode(s)

base64.b16decode(s[, casefold])

  

