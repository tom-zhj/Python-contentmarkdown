# python 多进程通信模块

多进程通信方法好多，不一而数。刚才试python封装好嘅多进程通信模块 multiprocessing.connection。

简单测试一下，效率还可以，应该系对socket封装，效率可以达到4krps，可以满足好多方面嘅需求啦。

附代码如下:

client

    
    
    
    #!/usr/bin/python
    # -*- coding: utf-8 -*-
    """ download - slave
    """
    __author__ = 'Zagfai'
    __license__ = 'MIT@2014-02'
     
    import webtul
    from multiprocessing.connection import Client
     
    a = 0
    try:
        while True:
            a += 1
            address = ('10.33.41.112', 6666)
            conn = Client(address, authkey='hellokey')
            #print conn.recv()
            d = conn.recv()
            conn.close()
    except:
        pass
    print a

server  

    
    
    
    #!/usr/bin/python
    # -*- coding: utf-8 -*-
    """ downloader - master server
    """
    __author__ = 'Zagfai'
    __license__ = 'MIT@2014-02'
     
    import webtul
    from multiprocessing.connection import Listener
    from threading import Thread
     
    def listener():
        address = ('10.33.41.112', 6666)
        listener = Listener(address, backlog=100, authkey='hellokey')
        while True:
            conn = listener.accept()
            #print 'connection accepted from', listener.last_accepted
            try:
                conn.send({'1':2, '2':'abc'})
            except Exception, e:
                print e
            finally:
                conn.close()
        listener.close()
     
    listener_th = Thread(target=listener)
    listener_th.daemon = True
    listener_th.start()
    listener_th.join(timeout=20)

  

