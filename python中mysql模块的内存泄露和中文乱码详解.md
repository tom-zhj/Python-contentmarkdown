# python中mysql模块的内存泄露和中文乱码详解

mysql-python的连接时，默认大家会写成

    
    
    con=MySQLdb.connect(user='xxx',passwd='xxx',host='xxx',port=6600,charset='gbk')

一旦指定了"gbk"，默认mysql-python会设定use_unicode=True。结果是mysql-python会利用python自己的
codec模块去做字符解码工作，但实际中发现mysql库gbk编码字符集比python的gbk编码集大。一些在mysql里可以存储的字符，拿
python的codec去解析就会抛错。更严重的问题是，在mysql-python1.2.3之前，use_unicode=True即让 mysql-
python解码这块存在内存泄露的bug。解码出来所有数据库字符串经过mysql-python出来都是unicode
object，要输出到文件需要再次编码。

  

解决方法是强制指定use_unicode=False。即：

    
    
    con=MySQLdb.connect(user='xxx',passwd='xxx',host='xxx',port=6600,charset='gbk',use_unicode=False)

这样既不会有内存泄露，也不需要在输出文件时进行编码。也回避了python的codec不能解析mysql gbk里面存放的字符串的问题。
最后对于mysql4，我们可以将charset参数留空：

    
    
    con=MySQLdb.connect(user='xxx',passwd='xxx',host='xxx',port=6600,use_unicode=False)

这样就完美解决了这个问题，哈哈

