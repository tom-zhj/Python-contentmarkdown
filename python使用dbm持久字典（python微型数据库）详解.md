# python使用dbm持久字典（python微型数据库）详解

在一些python小型应用程序中，不需要关系型数据库时，可以方便的用持久字典来存储名称/值对，它与python的字典非常类似，主要区别在于数据是在磁盘读取和
写入的。另一个区别在于dbm的键和值必须是字符串类型。

## 1.选择dbm模块

python支持很多dbm模块，遗憾的是，每个dbm模块创建的文件不兼容。

下表列出这些模块：

模块说明

dbm选择最好的dbm模块

dbm.dumb使用dbm库的一个简单但可移植的实现

dbm.gnu使用GNU dbm的库

一般除非某个dbm库有特殊高级功能，那就用dbm模块。

## 2.创建持久词典

    
    
    import dbm
    db = dbm.open('Bookmark', 'c')
    #添加选项
    db['MyBlog'] = 'jonathanlife.sinaapp.com'
    print(db['MyBlog'])
    #保存，关闭
    db.close()

open函数关于打开dbm的方式有三种：

标志用法

C打开文件对其读写，必要时创建该文件

W打开文件对其读写,如果文件不存在，不会创建它

N打开文件进行读写，但总是创建一个新的空白文件

也可以传递另一种表示模式的可选参数，该模式保存了一组UNIX文件权限，这里不细说。

3.访问持久字典

从open函数返回的对象视作一个字典对象。对值的存取方式如下：

    
    
    db[‘key’] = ‘value’
    value = db[‘key’]
    #删除值:
    del db[‘key’]
    #遍历所有key：
    for key in db.keys():
       #your code here

代码实例：

    
    
    import dbm
    #open existing file
    db = dbm.open('websites', 'w')
    #add item
    db['first_data'] = 'Hello world'
       
    #verify the previous item remains
    if db['first_data'] != None:
        print('the data exists')
    else:
        print('Missing item')
      
    #iterate over the keys, may be slow
    for key in db.keys():
        print("Key=",key," value=",db[key])
      
    #delete item
    del db['first_data']
      
    #close and save to disk
    db.close()

