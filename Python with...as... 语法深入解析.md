# Python with...as... 语法深入解析

with从Python 2.5就有，需要from __future__ import with_statement。自python
2.6开始，成为默认关键字。

也就是说with是一个控制流语句，跟if/for/while/try之类的是一类的，with可以用来简化try finally代码，看起来可以比try
finally更清晰。

    
    
    with EXPRESSION [ as VARIABLE] WITH-BLOCK

基本思想是with所求值的对象必须有一个__enter__()方法，一个__exit__()方法。

紧跟with后面的语句被求值后，返回对象的__enter__()方法被调用，这个方法的返回值将被赋值给as后面的变量。当with后面的代码块全部被执行完之后
，将调用前面返回对象的__exit__()方法。

  

with expresion as variable的执行过程是，首先执行__enter__函数，它的返回值会赋给as后面的variable，想让它返回什么
就返回什么，只要你知道怎么处理就可以了，如果不写as variable，返回值会被忽略。

然后，开始执行with-block中的语句，不论成功失败(比如发生异常、错误，设置sys.exit())，在with-
block执行完成后，会执行__exit__函数。

这样的过程其实等价于：

    
    
    try:
        执行 __enter__的内容
        执行 with_block.
    finally:
        执行 __exit__内容

  

再看个例子

    
    
    file = open("/tmp/foo.txt")
    try:
        data = file.read()
    finally:
        file.close()

使用with...as...的方式替换，修改后的代码是：

    
    
    with open("/tmp/foo.txt") as file:
        data = file.read()



    
    
    #!/usr/bin/env python
    # with_example01.py
     
    class Sample:
        def __enter__(self):
            print "In __enter__()"
            return "Foo"
     
        def __exit__(self, type, value, trace):
            print "In __exit__()"
            
    def get_sample():
        return Sample()
    with get_sample() as sample:
        print "sample:", sample

执行结果为

    
    
    In __enter__()
    sample: Foo
    In __exit__()

1\. __enter__()方法被执行

2\. __enter__()方法返回的值 - 这个例子中是"Foo"，赋值给变量'sample'

3\. 执行代码块，打印变量"sample"的值为 "Foo"

4\. __exit__()方法被调用with真正强大之处是它可以处理异常。可能你已经注意到Sample类的__exit__方法有三个参数- val,
type 和 trace。这些参数在异常处理中相当有用。

  

