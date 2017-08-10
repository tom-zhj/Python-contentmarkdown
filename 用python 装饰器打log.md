# 用python 装饰器打log


    #! /usr/bin/env python
    # coding=utf-8
     
    from time import time
    def logged(when):
        def log(f,*args,**kargs):
            print("called: function:%s,args:%r,kargs:%r"%(f,args,kargs))
        def pre_logged(f):
            def wrapper(*args,**kargs):
                log(f,*args,**kargs)
                return f(*args,**kargs)
        def post_logged(f):
            def wrapped(*args,**kargs):
                now=time()
                try:
                    return f(*args,**kargs)
                finally:
                    log(f,*args,**kargs)
                    print("time delta:%s"%(time()-now))
            return wrapped
        try:
            #从这里开始调用
            return{"pre":pre_logged,"post":post_logged}[when]
        except Exception as e:
            print(e)
     
    @logged("post")
    def hello(name):
        print("hello",name)
    @logged("post")
    def test(a,b=1):
        print(a+b)
     
    hello("world")
    test(1,2)

  

