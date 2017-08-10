# 使用python调用新浪微博接口

微博很火啊，那python是不是可以调用微博的api做一个小应用呢？答案是：必须可以，哈哈

    
    
    使用python调用weibo api
      
     # 调用的url地址  此为获取某人的个人信息的api  http://open.weibo.com/wiki/2/users/show
     the_url = 'https://api.weibo.com/2/users/show.json?uid=105729xxxx&access_token=2.xxx__YJBzk8g4Ddfd33f10237XXXXX'
      
     http_body = None
      
     # 发送请求并读取返回 返回的内容是真个html源代码，或者json数据，可以通过文件输出或者包一层repr()来查看内容
     req = urllib2.Request(the_url, data=http_body)
      
     #当然也可以用此来发送请求，并读取返回的内容是真个html源代码，可以通过文件输出或者包一层repr()来查看内容
     req = urllib2.Request("http://www.baidu.com", data=http_body)
      
     resp = urllib2.urlopen(req)
     print repr(resp.read())
      
      
      
     import json
     # 原配json工具
     json.loads(resp.read(), object_hook=_obj_hook)
      
      
     io = StringIO('["streaming API"]')
     print io.getvalue()
     # 输出为 ["streaming API"] io输出为一个StringIO对象 <cStringIO.StringI object at 0x1006aed80>
      
     # 注 json工具 提供load和loads2个方法
     json.load(fp[, encoding[, cls[, object_hook[, parse_float[, parse_int[, parse_constant[, object_pairs_hook[, **kw]]]]]]]])
      
     json.loads(s[, encoding[, cls[, object_hook[, parse_float[, parse_int[, parse_constant[, object_pairs_hook[, **kw]]]]]]]])
      
     其中fp 按照官方的解释[http://docs.python.org/2/glossary.html#term-file-object]是面向文件的系统，或者那种提供.read(),.write()方法的对象，比如上面的StringIO
     s 为普通的str或者unicode，相应json数据于python数据的对应如下
      
     [http://docs.python.org/2/library/json.html?highlight=object_hook#json-to-py-table]
     JSON    Python
     object    dict
     array    list
     string    unicode
     number (int)    int, long
     number (real)    float
     true    True
     false    False
     null    None
      
     # 转换json数据
     如果json数据如此
     {
         "ip_limit": 1000,
         "limit_time_unit": "HOURS",
         "remaining_ip_hits": 1000,
         "remaining_user_hits": 146,
         "reset_time": "2013-04-19 15:00:00",
         "reset_time_in_seconds": 3286,
         "user_limit": 150
     }
     # 关于 object_hook 非必选函数 用于替换原本钓鱼json.loads返回的一个dict，
     # json.loads返回的dict如果遇到一个不存在的量 比如"abc" 则会报错，
     # 于是用object_hook 返回一个自己定义的dict 让他在遇到"abc"这个不存在的变量时不直接反错
     # 而是try catch一下 返回None  更稳定
      
     def _parse_json(s):
         ' parse str into JsonDict '
      
         def _obj_hook(pairs):
             ' convert json object to python object '
             o = JsonDict()
             # print pairs['list_id']
             print 'text' in pairs
             if 'text' in pairs:
                 # it can output utf8
                 print pairs['text']
             # --- 
             for k, v in pairs.iteritems():
                 o[str(k)] = v
             return o
         #loads  is  different from load
         return json.loads(s, object_hook=_obj_hook)
      
     class JsonDict(dict):
         ' general json object that allows attributes to be bound to and also behaves like a dict '
      
         def __getattr__(self, attr):
             try:
                 return self[attr]
             except KeyError:
                 raise AttributeError(r"'JsonDict' object has no attribute '%s'" % attr)
      
         def __setattr__(self, attr, value):
             self[attr] = value
      
     # 此 JsonDict 是dict（字典对象）的子类 它重写了 取索引的方法 __getattr__ 和 __setattr__
      
     # 这里获取dict里的数据有个技巧 在获取JsonDict的对象时有2中方法
     d1 = JsonDict();
      
     d1['a'] = 'strra'
     print d1['a']
     print d1.get('a')
     # 此 aaaa 并不在dict中
     print d1.get('aaaa')   # None
     print d1['aaaa']   # key error~
      
     # 使用get获取dict里的数据 遇到空的数据的话 可以避免error的产生 自动转成None

  

