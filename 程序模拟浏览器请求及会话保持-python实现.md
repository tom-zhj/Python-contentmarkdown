# 程序模拟浏览器请求及会话保持-python实现

python下读取一个页面的数据可以通过urllib2轻松实现请求

    
    
    import urllib2
    print urllib2.urlopen('http://www.pythontab.com').read()

涉及到页面的POST请求操作的话需要提供头信息，提交的post数据和请求页面。

其中的post数据需要urllib.encode()一下，其实就是将字典转换成“data1=value1&data2=value2”的格式。

    
    
    import urllib
    import urllib2
     
    HEADER = {
        'User-Agent' : 'Mozilla/5.0 (Windows NT 6.1; WOW64; rv:31.0) Gecko/20100101 Firefox/31.0',
        'Referer' : 'http://202.206.1.163/logout.do'
    }
     
    POSTDATA = {
        'data1': 'value1',
        'data2': 'value2'
    }
     
    HOSTURL = 'http://xxx.com'
     
    enpostdata = urllib.urlencode(POSTDATA)
    urlrequest = urllib2.Request(hosturl,enpostdata,HEADER)
    urlresponse = urllib2.urlopen(urlrequest)
     
    print urlresponse.read()

请求之后浏览器会有一个会话保持的过程，会话都是保存在一个cookie里面的，下一次页面的请求会把cookie放到请求头，如果cookie丢失会话也就断开了。

在python下面需要设置一下cookie的保持

    
    
    # cookie set
    # 用来保持会话
    cj = cookielib.LWPCookieJar()
    cookie_support = urllib2.HTTPCookieProcessor(cj)
    opener = urllib2.build_opener(cookie_support, urllib2.HTTPHandler)
    urllib2.install_opener(opener)

下面是将以上知识点汇总写的一个库文件，方便使用：

    
    
    # filename: analogop.py
     
    #!/usr/bin/python
    # -*-coding:UTF-8 -*-
     
    # author: 初行
    # qq: 121866673
    # mail: zxbd1016@163.com
    # message: I need a python job
    # time: 2014/10/8
     
    import urllib
    import urllib2
    import cookielib
     
    # cookie set
    # 用来保持会话
    cj = cookielib.LWPCookieJar()
    cookie_support = urllib2.HTTPCookieProcessor(cj)
    opener = urllib2.build_opener(cookie_support, urllib2.HTTPHandler)
    urllib2.install_opener(opener)
     
    # default header
    HEADER = {
        'User-Agent' : 'Mozilla/5.0 (Windows NT 6.1; WOW64; rv:31.0) Gecko/20100101 Firefox/31.0',
        'Referer' : 'http://202.206.1.163/logout.do'
    }
     
    # operate method
    def geturlopen(hosturl, postdata = {}, headers = HEADER):
        # encode postdata
        enpostdata = urllib.urlencode(postdata)
        # request url
        urlrequest = urllib2.Request(hosturl, enpostdata, headers)
        # open url
        urlresponse = urllib2.urlopen(urlrequest)
        # return url
        return urlresponse

这个是测试文件，因为读者没有测试环境，需要自己搭建或者找个网站测试：

    
    
    #filename: test.py
    from analogop import geturlopen
     
    postd = {
        'usernum': '2011411111',
        'upw': '124569',
        'userip': '192.168.10.1',
        'token': 'xxx'
    }
     
    urlread = geturlopen('http://127.0.0.1:8000/login/', postd)
    print urlread.read().decode('utf-8')
    urlread = geturlopen('http://127.0.0.1:8000/chafen/', {})
    print urlread.read().decode('utf-8')

  

