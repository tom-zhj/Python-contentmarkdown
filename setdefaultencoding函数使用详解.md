# setdefaultencoding函数使用详解

sys.getdefaultencoding()是设置默认的string的编码格式，如果你在python中进行编码和解码的时候，不指定编码方式，那么pyth
on就会使用defaultencoding。

而python2.x的的defaultencoding是ascii,这也就是大多数python编码报错：“UnicodeDecodeError:
'ascii' codec can't decode byte ......”的原因。

与此有类似功能的# coding:utf-8 作用是定义源代码的编码. 如果没有定义, 此源码中是不可以包含中文字符串的.

注意：python2.7以后setdefaultencoding就废弃掉了，所以在python3.x中不可使用

  

代码实例：

    
    
    #!/usr/bin/env python    
    #encoding: utf-8  
    import sys   #引用sys模块进来，并不是进行sys的第一次加载  
    reload(sys)  #重新加载sys  
    sys.setdefaultencoding('utf8')  ##调用setdefaultencoding函数

可以正确的执行，可是下面的代码会出错

    
    
    #!/usr/bin/env python    
    #encoding: utf-8  
    import sys     
    sys.setdefaultencoding('utf8')

要在调用setdefaultencoding时必须要先reload一次sys模块，因为这里的import语句其实并不是sys的第一次导入语句，也就是说这里其
实可能是第二、三次进行sys模块的import，这里只是一个对sys的引用，只能reload才能进行重新加载。

  

那么为什么要重新加载，而直接引用过来则不能调用该函数呢？因为setdefaultencoding函数在被系统调用后被删除了，所以通过import引用进来时其
实已经没有了，所以必须reload一次sys模块，这样setdefaultencoding才会为可用，才能在代码里修改解释器当前的字符编码。

  

在python安装目录的Lib文件夹下，有一个叫site.py的文件，在里面可以找到main() --> setencoding()-->sys.setde
faultencoding(encoding),因为这个site.py每次启动python解释器时会自动加载，所以main函数每次都会被执行，setdefa
ultencoding函数一出来就已经被删除了。

  

