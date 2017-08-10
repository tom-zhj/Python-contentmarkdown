# python下redis安装和使用

**python下redis安装**  
  
用python操作redis数据库，先下载redis-py模块下载地址https://github.com/andymccurdy/redis-py  
  
shell# wget https://github.com/andymccurdy/redis-py

  
然后解压  


在解压目录运行 python setup.py install安装模块即可  
  
安装完成



**使用：**



    
    
    import redis
     
    r = redis.Redis(host=&rsquo;localhost&rsquo;, port=6379, db=0)
     
    r['test'] = &lsquo;test&rsquo; #或者可以r.set(&lsquo;test&rsquo;, &lsquo;test&rsquo;) 设置key
     
    r.get(&lsquo;test&rsquo;)  #获取test的值
     
    r.delete(&lsquo;test&rsquo;) #删除这个key
     
    r.flushdb() #清空数据库
     
    r.keys() #列出所有key
     
    r.exists(&lsquo;test&rsquo;) #检测这个key是否存在
     
    r.dbsize() #数据库中多少个条数

  

