# curl库pycurl实例及参数详解

pycurl是功能强大的python的url库，是用c语言写的，速度很快，比urllib和httplib都快。

今天我们来看一下pycurl的用法及参数详解

常用方法：

pycurl.Curl() #创建一个pycurl对象的方法

pycurl.Curl().setopt(pycurl.URL, http://www.pythontab.com) #设置要访问的URL

pycurl.Curl().setopt(pycurl.MAXREDIRS, 5) #设置最大重定向次数

pycurl.Curl().setopt(pycurl.CONNECTTIMEOUT, 60)

pycurl.Curl().setopt(pycurl.TIMEOUT, 300) #连接超时设置

pycurl.Curl().setopt(pycurl.USERAGENT, "Mozilla/5.0 (compatible; MSIE 6.0;
Windows NT 5.1; SV1; .NET CLR 1.1.4322)") #模拟浏览器

pycurl.Curl().perform() #服务器端返回的信息

pycurl.Curl().getinfo(pycurl.HTTP_CODE) #查看HTTP的状态 类似urllib中status属性

pycurl.NAMELOOKUP_TIME 域名解析时间

pycurl.CONNECT_TIME 远程服务器连接时间

pycurl.PRETRANSFER_TIME 连接上后到开始传输时的时间

pycurl.STARTTRANSFER_TIME 接收到第一个字节的时间

pycurl.TOTAL_TIME 上一请求总的时间

pycurl.REDIRECT_TIME 如果存在转向的话，花费的时间

pycurl.EFFECTIVE_URL

pycurl.HTTP_CODE HTTP 响应代码

pycurl.REDIRECT_COUNT 重定向的次数

pycurl.SIZE_UPLOAD 上传的数据大小

pycurl.SIZE_DOWNLOAD 下载的数据大小

pycurl.SPEED_UPLOAD 上传速度

pycurl.HEADER_SIZE 头部大小

pycurl.REQUEST_SIZE 请求大小

pycurl.CONTENT_LENGTH_DOWNLOAD 下载内容长度

pycurl.CONTENT_LENGTH_UPLOAD 上传内容长度

pycurl.CONTENT_TYPE 内容的类型

pycurl.RESPONSE_CODE 响应代码

pycurl.SPEED_DOWNLOAD 下载速度

pycurl.SSL_VERIFYRESULT

pycurl.INFO_FILETIME 文件的时间信息

pycurl.HTTP_CONNECTCODE HTTP 连接代码

pycurl.HTTPAUTH_AVAIL

pycurl.PROXYAUTH_AVAIL

pycurl.OS_ERRNO

pycurl.NUM_CONNECTS

pycurl.SSL_ENGINES

pycurl.INFO_COOKIELIST

pycurl.LASTSOCKET

pycurl.FTP_ENTRY_PATH

  

实例：

    
    
    import StringIO
    import pycurl
     
    c = pycurl.Curl()
    str = StringIO.StringIO()
    c.setopt(pycurl.URL, "http://www.pythontab.com")
    c.setopt(pycurl.WRITEFUNCTION, str.write)
    c.setopt(pycurl.FOLLOWLOCATION, 1)
     
    c.perform()
    print c.getinfo(pycurl.EFFECTIVE_URL)

熟悉php的朋友可能已经发现了， 这个curl库的使用方法非常类似于php的curl。

