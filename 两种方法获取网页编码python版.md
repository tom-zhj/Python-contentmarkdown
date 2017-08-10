# 两种方法获取网页编码python版

在web开发的时候我们经常会遇到网页抓取和分析，各种语言都可以完成这个功能。我喜欢用python实现，因为python提供了很多成熟的模块，可以很方便的实现
网页抓取。

但是在抓取过程中会遇到编码的问题，那今天我们来看一下如何判断网页的编码：

网上很多网页的编码格式都不一样，大体上是GBK,GB2312，UTF-8等。

我们在获取网页的的数据后，先要对网页的编码进行判断，才能把抓取的内容的编码统一转换为我们能够处理的编码，避免乱码问题的出现。

  

下面介绍**两种判断网页编码的方法**：

## 方法一：使用urllib模块的getparam方法

    
    
    import urllib
     
    fopen1 = urllib.urlopen('http://www.baidu.com').info()
     
    fopen2 = urllib.urlopen('http://www.pythontab.com').info()
     
    print fopen1.getparam('charset')# baidu
     
    print fopen2.getparam('charset')# pythontab

  

执行结果为：

gbk

None

  

PS: 呵呵，其实，上面的获取的编码都是不正确的，我们可以自己打开网页查看源代码，发现baidu的是gb2312，而pythontab是utf-8。唉，这个
方法确实有点坑爹啊。检测不准确、检测不到，它都占了，所以很不靠谱，下面介绍一个靠谱的方法。

  

## 方法二：使用chardet模块

    
    
    #如果你的python没有安装chardet模块，你需要首先安装一下chardet判断编码的模块哦
    import chardet 
    import urllib
    #先获取网页内容
    data = urllib.urlopen('http://www.pythontab.com').read()
    #用chardet进行内容分析
    chardit = chardet.detect(data)
     
    data1 = urllib.urlopen('http://www.baidu.com').read()
     
    chardit1 = chardet.detect(data1)
     
    print chardit['encoding'] # pythontab
     
    print chardit1['encoding'] # baidu

  

执行结果为：

utf-8

gb2312

  

这两个结果都是正确的哦，各位可以去亲自验证一下~~

  

总结：第二个方法很准确，在网页编码分析的时候用python模块分析内容是最准确的，而使用分析meta头信息的方法是不太准确的。

  

