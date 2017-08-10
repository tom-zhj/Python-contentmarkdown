# 分别用python2和python3伪装浏览器爬取网页内容

python网页抓取功能非常强大，使用urllib或者urllib2可以很轻松的抓取网页内容。但是很多时候我们要注意，可能很多网站都设置了防采集功能，不是那
么轻松就能抓取到想要的内容。

今天我来分享下载python2和python3中都是如何来模拟浏览器来跳过屏蔽进行抓取的。

## 最基础的抓取：

    
    
    #! /usr/bin/env python
    # -*- coding=utf-8 -*-
    # @Author pythontab
    import urllib.request
    url = "http://www.pythontab.com"
    html = urllib.request.urlopen(url).read(）
    print(html)

  

但是...有些网站不能抓取，进行了防采集设置，所以我们要变换一下方法

## python2中（最新稳定版本python2.7）

    
    
    #! /usr/bin/env python
    # -*- coding=utf-8 -*- 
    # @Author pythontab.com
    import urllib2
    url="http://pythontab.com"
    req_header = {'User-Agent':'Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.11 (KHTML, like Gecko) Chrome/23.0.1271.64 Safari/537.11',
                 'Accept':'text/html;q=0.9,*/*;q=0.8',
                 'Accept-Charset':'ISO-8859-1,utf-8;q=0.7,*;q=0.3',
                 'Accept-Encoding':'gzip',
                 'Connection':'close',
                 'Referer':None #注意如果依然不能抓取的话，这里可以设置抓取网站的host
                 }
    req_timeout = 5
    req = urllib2.Request(url,None,req_header)
    resp = urllib2.urlopen(req,None,req_timeout)
    html = resp.read()
    print(html)

## python3中（最新稳定版本python3.3）  

    
    
    #! /usr/bin/env python
    # -*- coding=utf-8 -*- 
    # @Author pythontab
    import urllib.request
     
    url = "http://www.pythontab.com"
    headers = {'User-Agent':'Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.11 (KHTML, like Gecko) Chrome/23.0.1271.64 Safari/537.11',
                 'Accept':'text/html;q=0.9,*/*;q=0.8',
                 'Accept-Charset':'ISO-8859-1,utf-8;q=0.7,*;q=0.3',
                 'Accept-Encoding':'gzip',
                 'Connection':'close',
                 'Referer':None #注意如果依然不能抓取，这里可以设置抓取网站的host
                 }
     
    opener = urllib.request.build_opener()
    opener.addheaders = [headers]
    data = opener.open(url).read()
    print(data)

  

