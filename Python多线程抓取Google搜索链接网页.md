# Python多线程抓取Google搜索链接网页

1）urllib2+BeautifulSoup抓取Goolge搜索链接

近期，参与的项目需要对Google搜索结果进行处理，之前学习了Python处理网页相关的工具。实际应用中，使用了urllib2和beautifulsoup来
进行网页的抓取，但是在抓取google搜索结果的时候，发现如果是直接对google搜索结果页面的源代码进行处理，会得到很多“脏”链接。

看下图为搜索“titanic  james”的结果：

![QQ截图20130407145449](http://www.pythontab.com/uploadfile/2013/0410/2013041009
5957782.jpg)  

图中红色标记的是不需要的，蓝色标记的是需要抓取处理的。

这种“脏链接”当然可以通过规则过滤的方法来过滤掉，但是这样程序的复杂度就高了。正当自己愁眉苦脸的正在写过滤规则时。同学提醒说google应该提供相关的api
，才恍然大明白。

（2）Google Web Search API+多线程

文档中给出使用Python进行搜索的例子：

    
    
    import simplejson 
       
    # The request also includes the userip parameter which provides the end  
    # user's IP address. Doing so will help distinguish this legitimate  
    # server-side traffic from traffic which doesn't come from an end-user.  
    url = ('https://ajax.googleapis.com/ajax/services/search/web'
           '?v=1.0&q=Paris%20Hilton&userip=USERS-IP-ADDRESS') 
       
    request = urllib2.Request( 
        url, None, {'Referer': /* Enter the URL of your site here */}) 
    response = urllib2.urlopen(request) 
       
    # Process the JSON string.  
    results = simplejson.load(response) 
    # now have some fun with the results... 
     
    import simplejson
      
    # The request also includes the userip parameter which provides the end
    # user's IP address. Doing so will help distinguish this legitimate
    # server-side traffic from traffic which doesn't come from an end-user.
    url = ('https://ajax.googleapis.com/ajax/services/search/web'
           '?v=1.0&q=Paris%20Hilton&userip=USERS-IP-ADDRESS')
      
    request = urllib2.Request(
        url, None, {'Referer': /* Enter the URL of your site here */})
    response = urllib2.urlopen(request)
      
    # Process the JSON string.
    results = simplejson.load(response)
    # now have some fun with the results..

  

实际应用中可能需要抓取google的很多网页，所以还需要使用多线程来分担抓取任务。使用google web search
api的参考详细介绍，请看此处（这里介绍了Standard URL
Arguments）。另外要特别注意，url中参数rsz必须是8（包括8）以下的值，若大于8，会报错的！

（3）代码实现

代码实现还存在问题，但是能够运行，鲁棒性差，还需要进行改进，希望各路大神指出错误（初学Python），不胜感激。

    
    
    #-*-coding:utf-8-*-  
    import urllib2,urllib 
    import simplejson 
    import os, time,threading 
      
    import common, html_filter 
    #input the keywords  
    keywords = raw_input('Enter the keywords: ')                                  
      
    #define rnum_perpage, pages  
    rnum_perpage=8
    pages=8                        
      
    #定义线程函数  
    def thread_scratch(url, rnum_perpage, page): 
     url_set = []  
     try: 
       request = urllib2.Request(url, None, {'Referer': 'http://www.sina.com'}) 
       response = urllib2.urlopen(request) 
       # Process the JSON string.  
       results = simplejson.load(response) 
       info = results['responseData']['results'] 
     except Exception,e: 
       print 'error occured'
       print e 
     else: 
       for minfo in info: 
          url_set.append(minfo['url']) 
          print minfo['url'] 
      #处理链接  
     i = 0
     for u in url_set: 
       try: 
         request_url = urllib2.Request(u, None, {'Referer': 'http://www.sina.com'}) 
         request_url.add_header( 
         'User-agent', 
         'CSC'
         ) 
         response_data = urllib2.urlopen(request_url).read() 
         #过滤文件  
         #content_data = html_filter.filter_tags(response_data)  
         #写入文件  
         filenum = i+page 
         filename = dir_name+'/related_html_'+str(filenum) 
         print '  write start: related_html_'+str(filenum) 
         f = open(filename, 'w+', -1) 
         f.write(response_data) 
         #print content_data  
         f.close() 
         print '  write down: related_html_'+str(filenum) 
       except Exception, e: 
         print 'error occured 2'
         print e 
       i = i+1
     return 
      
    #创建文件夹  
    dir_name = 'related_html_'+urllib.quote(keywords) 
    if os.path.exists(dir_name): 
       print 'exists  file'
       common.delete_dir_or_file(dir_name) 
    os.makedirs(dir_name) 
      
    #抓取网页  
    print 'start to scratch web pages:'
    for x in range(pages): 
      print "page:%s"%(x+1) 
      page = x * rnum_perpage 
      url = ('https://ajax.googleapis.com/ajax/services/search/web'
                      '?v=1.0&q=%s&rsz=%s&start=%s') % (urllib.quote(keywords), rnum_perpage,page) 
      print url 
      t = threading.Thread(target=thread_scratch, args=(url,rnum_perpage, page)) 
      t.start() 
    #主线程等待子线程抓取完  
    main_thread = threading.currentThread() 
    for t in threading.enumerate(): 
      if t is main_thread: 
        continue
      t.join() 

  

