# python批量抓取美女图片

学了python以后，知道python的抓取功能其实是非常强大的，当然不能浪费，呵呵。我平时很喜欢美女图，呵呵，程序员很苦闷的，看看美女，养养眼，增加点乐趣
。好，那就用python写一个美女图自动抓取程序吧~~  

其中用到urllib2模块和正则表达式模块。下面直接上代码：

**用python批量抓取美女图片**
    
    
    #!/usr/bin/env python
    #-*- coding: utf-8 -*-
    #通过urllib(2)模块下载网络内容
    import urllib,urllib2,gevent
    #引入正则表达式模块，时间模块
    import re,time
    from gevent import monkey
      
    monkey.patch_all()
      
    def geturllist(url):
        url_list=[]
        print url        
        s = urllib2.urlopen(url)
        text = s.read()
        #正则匹配，匹配其中的图片
        html = re.search(r'<ol.*</ol>', text, re.S)
        urls = re.finditer(r'<p><img src="(.+?)jpg" /></p>',html.group(),re.I)
        for i in urls:
            url=i.group(1).strip()+str("jpg")
            url_list.append(url)
        return url_list
      
    def download(down_url):
        name=str(time.time())[:-3]+"_"+re.sub('.+?/','',down_url)
        print name
        urllib.urlretrieve(down_url, "D:\\TEMP\\"+name)
      
    def getpageurl():
        page_list = []
        #进行列表页循环
        for page in range(1,700):
            url="http://jandan.net/ooxx/page-"+str(page)+"#comments"
            #把生成的url加入到page_list中
            page_list.append(url)
        print page_list
        return page_list
    if __name__ == '__main__':
        jobs = []
        pageurl = getpageurl()[::-1]
        #进行图片下载 
        for i in pageurl:
            for (downurl) in geturllist(i):
                jobs.append(gevent.spawn(download, downurl))
        gevent.joinall(jobs)

程序不长才45行，不是太难，大家可以研究下，这里我只是抛砖引玉，大家可以根据原理开发出其他的抓取程序，呵呵，自己想去吧。。。我就不多说了~~

