# python2.7 跨文件全局变量的方法

在使用Python编写的应用的过程中，有时会遇到多个文件之间传递同一个全局变量的情况。

文件1：globalvar.py

    
    
    #!/usr/bin/env python2.7 
    class GlobalVar: 
    db_handle = None 
    mq_client = None 
    def set_db_handle(db): 
    GlobalVar.db_handle = db 
    def get_db_handle(): 
    return GlobalVar.db_handle 
    def set_mq_client(mq_cli): 
    GlobalVar.mq_client = mq_cli 
    def get_mq_client(): 
    return GlobalVar.mq_client

文件2：set.py

    
    
    import globalvar as GlobalVar 
    def set(): 
    GlobalVar.set_mq_client(10) 
    print "------set mq_client in set.py------mq_client: " + str(GlobalVar.get_mq_client())

文件3：get.py

    
    
    #!/usr/bin/env python2.7 
    import globalvar as GlobalVar 
    def get(): 
    print "------get mq_client in get.py------mq_client: " + str(GlobalVar.get_mq_client())

文件4：main.py

    
    
    #!/usr/bin/env python2.7 
    import set 
    import get 
    set.set() 
    get.get()

其中globalvar.py中定义了两个全局变量，在set.py中的set函数中对其进行赋值，在get.py文件中的get函数取值并打印。main.py函数
作为应用入口，调用set和get。

这样就可以看到一个完整的应用中，全局变量的跨文件使用。

  

