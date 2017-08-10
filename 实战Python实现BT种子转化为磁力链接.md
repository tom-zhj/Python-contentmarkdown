# 实战Python实现BT种子转化为磁力链接

经常看电影的朋友肯定对BT种子并不陌生，但是BT种子文件相对磁力链来说存储不方便，而且在网站上存放BT文件容易引起版权纠纷，而磁力链相对来说则风险小一些。

将BT种子转换为占用空间更小，分享更方便的磁力链还是有挺大好处的。

今天咱们来看下如何将种子转换成磁力链接，方案是：利用python的bencode模块，用起来比较简单

首先要安装这个模块，安装命令：

    
    
    pip install bencode

如果没有安装pip，请移步《[详解python包管理器pip安装](http://www.pythontab.com/html/2015/pythonhex
inbiancheng_1012/963.html)》

### 实战代码

安装完成后，我们来看下代码：

系统环境：Linux

Python环境：Python2.7

请注意python版本

bt2url.py

    
    
    #! /usr/local/bin/python
    # @desc python通过BT种子生成磁力链接 
    # @date 2015/11/10
    # @author pythontab.com
    import bencode
    import sys
    import hashlib
    import base64
    import urllib
    #获取参数
    torrentName = sys.argv[1]
    #读取种子文件
    torrent = open(torrentName, 'rb').read()
    #计算meta数据
    metadata = bencode.bdecode(torrent)
    hashcontents = bencode.bencode(metadata['info'])
    digest = hashlib.sha1(hashcontents).digest()
    b32hash = base64.b32encode(digest)
    #打印
    print 'magnet:?xt=urn:btih:%s' % b32hash

### 如何使用？

命令：  

    
    
    python bt2url.py test.torrent

结果：

    
    
    magnet:?xt=urn:btih:MWXFHXOGE2UMR7WBFZYEJPM3LF2VIHNH

  

