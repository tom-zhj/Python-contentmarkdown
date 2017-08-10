# Python 如何将一变量做为函数名？

PHP 有变量函数这一用法（见 http://www.php.net/manual/en/functions.variable-functions.php）

即有一字符串变量

现在想用这个变量的值做为某函数名来调用

代码如下：

    
    
    def bar():
        return 'bar'
    foo = 'bar'
    foo() #此处想调用 bar()

不知道 Python 是否有此类用法？

哈哈，python是这么强大，python当然也是可以支持的

python有个eval函数（貌似好多语言都有这个函数）可以解决这个问题。

代码如下：

    
    
    def foo():
      print 'hi'
    t = eval('foo')
    t()

问题二：python可以import一个为某变量值的模块吗？

    
    
    foo = 'bar'
    import foo #此处想导入 bar

这个地方要用到另外一个函数了。。。

上代码：

    
    
    m = 'sys'
    exec "import " +　m

这样就ok了

  

