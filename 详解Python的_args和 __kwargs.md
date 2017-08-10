# 详解Python的*args和 **kwargs

*args表示任何多个无名参数，它是一个tuple；**kwargs表示关键字参数，它是一个dict。
    
    
    def fun(*args, **kwargs):
        print 'args = ', args
        print 'kwargs = ', kwargs
        print '###'
    if __name__ == '__main__':
        foo(1,2,3,4)
        foo(a=1,b=2,c=3)
        foo(1,2,3,4, a=1,b=2,c=3)
        foo('a', 1, None, a=1, b='2', c=3)

输出结果如下：  

    
    
    args =  (1, 2, 3, 4) 
    kwargs =  {} 
    ###
    args =  () 
    kwargs =  {'a': 1, 'c': 3, 'b': 2} 
    ###
    args =  (1, 2, 3, 4) 
    kwargs =  {'a': 1, 'c': 3, 'b': 2} 
    ###
    args =  ('a', 1, None) 
    kwargs =  {'a': 1, 'c': 3, 'b': '2'} 
    ###

  

可以看到，这两个是python中的可变参数。

注意：同时使用*args和**kwargs时，必须*args参数列要在**kwargs前，像foo(a=1, b='2', c=3, a', 1,
None, )这样调用的话，会提示语法错误“SyntaxError: non-keyword arg after keyword arg”。

    
    
    def fun2(param1, *args, **kwargs):
        print 'param1 = ', param1
        print 'args = ', args
        print 'kwargs = ', kwargs
        print '###'
    fun2(1, 2, 3, 4, a=1,b=2,c=3)

输出结果：

    
    
    param1 =  1
    args =  (2,3,4)
    kwargs =  {'a': 1, 'c': 3, 'b': 2} 
    ###

1赋值给了param1， 剩下的2，3，4给了*args， 其他的都赋值给了**kwargs

  

还有一个很漂亮的用法，就是创建字典：

    
    
    def kw_dict(**kwargs):
        return kwargs
    print kw_dict(a=1,b=2,c=3)

结果：

    
    
    {'a':1, 'b':2, 'c':3}

其实python中就带有dict类，使用dict(a=1,b=2,c=3)即可创建一个字典了。

  

