# Python 中的POST／GET包构建以及随机字符串的生成

现在，我们来用Python，创建GET包和POST包。

至于有什么用处，大家慢慢体会。

Python 中包含了大量的库，作为一门新兴的语言，Python 对HTTP有足够强大的支持。

现在，我们引入新的库 httplib 以及 urllib

这两个库根据名称，我们可以知道他们是对于HTTP以及URL的操作。

首先我们先要与服务器建立连接。（我们以某微博作为例子实现下文的各种功能）

conn = httplib.HTTPConnection("ti50*****com");

只要没有提示错误，我们就可以认为连接已成功，下面就可以进行数据包发送了。

在上文中我们说过了GET包的结构，只有HEARDER 部分。而在httplib中，heaer 是通过一个字典来保存的。下面我们来定义它：

headers = {"Content-Type": "application/x-www-form-urlencoded",

  "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",

  "Referer": "http://ti50.*****com/g/s?sid=*********************",

  "Accept-Encoding": "",

  "Accept-Language": "zh-CN,zh;q=0.8",

  "Accept-Charset": "GBK,utf-8;q=0.7,*;q=0.3",

  "Cookie": Cookie  }

Accept-Encoding 我们删除了其内容，这对于服务器来说我们客户端不能接受任何压缩的格式，数据包将用原始数据发送回来，这样我们就可以省去解压缩的过
程直接分析网页了，但是这样做的后果是流量大，网络实时性差。关于解压缩自然有别的库来专门处理。

然后我们可以直接发送了。

conn.request(method="GET",url='''http://ti50****com/g/s?*********_TK9EH&r='''
+ go_num + '''&aid=amsg&bid=******=true&ifh=1&ngpd=false''',headers=headers);

method 字段说明是发送何种类型的数据包。

url 字段以字符串的形式定义地址

header 字段定义包头。

一般来说，一个数据包发送至服务器，服务器会相应的返回一个应答包。而且这个应答包对于我们往往是有用的，我们用下面的命令获取应答包。

response = conn.getresponse();

对于上面这条语句中的括号，其表示读取应答包的前多少个字符。

POST包与GET包的创建过程基本相同。

只是我们需要新定义BODY，这个部分可以用字符串的方式进行定义。

params = 'msg=***************************'

我们仍然需要先于服务器进行连接。

conn = httplib.HTTPConnection("ti50*****com");

发送

conn.request(method="POST",url='''/g/s?sid=******************&ngpd=false''',bo
dy=params,headers=headers);

可以发现上面的这个公式和发送GET包的格式略有差距。

method 改变了。

url 里面没有写域名。

多了一个body 字段。

其中第二条可以想到，如果没定义域名，则系统将最近一次与服务器的连接用的域名进行替换。

获取应答包的方式与GET包相同。

乱七八糟的小应用。

（一） 随机字符串的生成。

当我们用POST做一些很有趣的事情时，常常会遇到服务器验证神马的，有时候我们可以用随机字符串来处理这样的情况。

python 中给了随机数的库…… random。

对于简单的应用非常方便。例如我们产生a与b 之间的一个随机整数。

random.randint(a,b)

>>> random.randint(10,20)

>>> 15

知道了这步，我们可以很简单的编写一个随机字符串的程序了，

    
    
    from random import Random 
    def random_str(randomlength): 
        str = '' 
        chars = 'AaBbCcDdEeFfGgHhIiJjKkLlMmNnOoPpQqRrSsTtUuVvWwXxYyZz0123456789'
        length = len(chars) - 1
        random = Random() 
        for i in range(randomlength): 
            str+=chars[random.randint(0, length)] 
        return str

  

显然当调用此函数时应该给出随机字符串长度。

当然，我们也可以通过修改chars中的字符来定义随机字符串中的字符。

（二） 程序运行时间

我们现在给出一个非常不精确的程序时间计算方法，

    
    
    from time import clock as now
    start = now()
    finish = now()
    run_time = finish - start
    print run_time

  

