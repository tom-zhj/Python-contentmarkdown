# Python 程序员经常犯的 10 个错误

关于Python

Python是一种解释性、面向对象并具有动态语义的高级程序语言。它内建了高级的数据结构，结合了动态类型和动态绑定的优点，这使得它在快速应用开发中非常有吸引力
，并且可作为脚本或胶水语言来连接现有的组件或服务。Python支持模块和包，从而鼓励了程序的模块化和代码重用。

关于这篇文章

Python简单易学的语法可能会使Python开发者–尤其是那些编程的初学者–忽视了它的一些微妙的地方并低估了这门语言的能力。

有鉴于此，本文列出了一个“10强”名单，枚举了甚至是高级Python开发人员有时也难以捕捉的错误。

### 常见错误 1: 滥用表达式作为函数参数的默认值

Python允许为函数的参数提供默认的可选值。尽管这是语言的一大特色，但是它可能会导致一些易变默认值的混乱。例如，看一下这个Python函数的定义：

    
    
    >>> def foo(bar=[]):        # bar is optional and defaults to [] if not specified  
    ...    bar.append("baz")    # but this line could be problematic, as we'll see...  
    ...    return bar

一个常见的错误是认为在函数每次不提供可选参数调用时可选参数将设置为默认指定值。在上面的代码中，例如，人们可能会希望反复（即不明确指定bar参数）地调用foo
()时总返回'baz'，由于每次foo()调用时都假定（不设定bar参数）bar被设置为[]（即一个空列表）。

但是让我们看一下这样做时究竟会发生什么：

    
    
    >>>&nbsp;foo()  
    ["baz"]>>>&nbsp;foo()  
    ["baz",&nbsp;"baz"]>>>&nbsp;foo()  
    ["baz",&nbsp;"baz",&nbsp;"baz"]

耶？为什么每次foo()调用时都要把默认值"baz"追加到现有列表中而不是创建一个新的列表呢？

答案是函数参数的默认值只会评估使用一次—在函数定义的时候。因此，bar参数在初始化时为其默认值（即一个空列表），即foo()首次定义的时候，但当调用foo(
)时（即，不指定bar参数时）将继续使用bar原本已经初始化的参数。

下面是一个常见的解决方法：

    
    
    >>> def foo(bar=None):  
    ...    if bar is None:        # or if not bar:  
    ...        bar = []  
    ...    bar.append("baz")  
    ...    return bar  
    ...  
    >>> foo()  
    ["baz"]  
    >>> foo()  
    ["baz"]  
    >>> foo()  
    ["baz"]

### 常见错误 2: 错误地使用类变量

考虑一下下面的例子：

    
    
    >>> class A(object):  
    ...     x = 1 
    ...  
    >>> class B(A):  
    ...     pass 
    ...  
    >>> class C(A):  
    ...     pass 
    ...  
    >>> print A.x, B.x, C.x  
    1 1 1

常规用一下。

    
    
    >>> B.x = 2 
    >>> print A.x, B.x, C.x  
    1 2 1

嗯，再试一下也一样。

    
    
    >>> A.x = 3 
    >>> print A.x, B.x, C.x  
    3 2 3

什么 $%#!&?? 我们只改了A.x，为什么C.x也改了?

在Python中，类变量在内部当做字典来处理，其遵循常被引用的方法解析顺序（MRO）。所以在上面的代码中，由于class
C中的x属性没有找到，它会向上找它的基类（尽管Python支持多重继承，但上面的例子中只有A）。换句话说，class
C中没有它自己的x属性，其独立于A。因此，C.x事实上是A.x的引用。

### 常见错误 3: 为 except 指定错误的参数

假设你有如下一段代码：

    
    
    >>> try:  
    ...     l = ["a", "b"]  
    ...     int(l[2])  
    ... except ValueError, IndexError:  # To catch both exceptions, right?  
    ...     pass 
    ...  
    Traceback (most recent call last):  
      File "<stdin>", line 3, in <module>  
    IndexError: list index out of range

这里的问题在于 except 语句并不接受以这种方式指定的异常列表。相反，在Python 2.x中，使用语法 except Exception, e
是将一个异常对象绑定到第二个可选参数（在这个例子中是 e）上，以便在后面使用。所以，在上面这个例子中，IndexError
这个异常并不是被except语句捕捉到的，而是被绑定到一个名叫 IndexError的参数上时引发的。

在一个except语句中捕获多个异常的正确做法是将第一个参数指定为一个含有所有要捕获异常的元组。并且，为了代码的可移植性，要使用as关键词，因为Python
2 和Python 3都支持这种语法：

    
    
    >>> try:  
    ...     l = ["a", "b"]  
    ...     int(l[2])  
    ... except (ValueError, IndexError) as e:    
    ...     pass 
    ...  
    >>>

### 常见错误 4:  不理解Python的作用域

Python是基于 LEGB 来进行作用于解析的, LEGB 是 Local, Enclosing, Global, Built-in
的缩写。看起来“见文知意”，对吗？实际上，在Python中还有一些需要注意的地方，先看下面一段代码：

    
    
    >>> x = 10 
    >>> def foo():  
    ...     x += 1 
    ...     print x  
    ...  
    >>> foo()  
    Traceback (most recent call last):  
      File "<stdin>", line 1, in <module>  
      File "<stdin>", line 2, in foo  
    UnboundLocalError: local variable 'x' referenced before assignment

这里出什么问题了？

上面的问题之所以会发生是因为当你给作用域中的一个变量赋值时，Python 会自动的把它当做是当前作用域的局部变量，从而会隐藏外部作用域中的同名变量。

很多人会感到很吃惊，当他们给之前可以正常运行的代码的函数体的某个地方添加了一句赋值语句之后就得到了一个 UnboundLocalError 的错误。
(你可以在这里了解到更多)

尤其是当开发者使用 lists 时，这个问题就更加常见.  请看下面这个例子：

    
    
    >>> lst = [1, 2, 3]  
    >>> def foo1():  
    ...     lst.append(5)   # 没有问题...  
    ...  
    >>> foo1()  
    >>> lst  
    [1, 2, 3, 5]  
     
    >>> lst = [1, 2, 3]  
    >>> def foo2():  
    ...     lst += [5]      # ... 但是这里有问题!  
    ...  
    >>> foo2()

  

Traceback (most recent call last):

  File "<stdin>", line 1, in <module>

  File "<stdin>", line 2, in foo

UnboundLocalError: local variable 'lst' referenced before assignment

嗯？为什么 foo2 报错，而foo1没有问题呢？

原因和之前那个例子的一样，不过更加令人难以捉摸。foo1 没有对 lst 进行赋值操作，而 foo2 做了。要知道， lst += [5] 是 lst =
lst + [5] 的缩写，我们试图对 lst 进行赋值操作（Python把他当成了局部变量）。此外，我们对 lst 进行的赋值操作是基于 lst
自身（这再一次被Python当成了局部变量），但此时还未定义。因此出错！

### 常见错误 5：当迭代时修改一个列表（List）

下面代码中的问题应该是相当明显的：

    
    
    >>> odd = lambda x : bool(x % 2)  
    >>> numbers = [n for n in range(10)]  
    >>> for i in range(len(numbers)):  
    ...     if odd(numbers[i]):  
    ...         del numbers[i]  # BAD: Deleting item from a list while iterating over it  
    ...

Traceback (most recent call last):

        File "<stdin>", line 2, in <module>

IndexError: list index out of range

当迭代的时候，从一个 列表 （List）或者数组中删除元素，对于任何有经验的开发者来说，这是一个众所周知的错误。尽管上面的例子非常明显，但是许多高级开发者在
更复杂的代码中也并非是故意而为之的。

幸运的是，Python包含大量简洁优雅的编程范例，若使用得当，能大大简化和精炼代码。这样的好处是能得到更简化和更精简的代码，能更好的避免程序中出现当迭代时修
改一个列表（List）这样的bug。一个这样的范例是递推式列表（list comprehensions）。而且，递推式列表（list
comprehensions）针对这个问题是特别有用的，通过更改上文中的实现，得到一段极佳的代码：

    
    
    >>> odd = lambda x : bool(x % 2)  
    >>> numbers = [n for n in range(10)]  
    >>> numbers[:] = [n for n in numbers if not odd(n)]  # ahh, the beauty of it all  
    >>> numbers  
    [0, 2, 4, 6, 8]

### 常见错误 6: 不明白Python在闭包中是如何绑定变量的

看下面这个例子：

    
    
    >>> def create_multipliers():  
    ...     return [lambda x : i * x for i in range(5)]  
    >>> for multiplier in create_multipliers():  
    ...     print multiplier(2)  
    ...

你也许希望获得下面的输出结果：

0

2

4

6

8

但实际的结果却是：

8

8

8

8

8

惊讶吧!

这之所以会发生是由于Python中的“后期绑定”行为——闭包中用到的变量只有在函数被调用的时候才会被赋值。所以，在上面的代码中，任何时候，当返回的函数被调用
时，Python会在该函数被调用时的作用域中查找 i 对应的值（这时，循环已经结束，所以 i 被赋上了最终的值——4）。

解决的方法有一点hack的味道：

    
    
    >>> def create_multipliers():  
    ...     return [lambda x, i=i : i * x for i in range(5)]  
    ...  
    >>> for multiplier in create_multipliers():  
    ...     print multiplier(2)  
    ...

0

2

4

6

8

在这里，我们利用了默认参数来生成一个匿名的函数以便实现我们想要的结果。有人说这个方法很巧妙，有人说它难以理解，还有人讨厌这种做法。但是，如果你是一个
Python 开发者，理解这种行为很重要。

### 常见错误 7: 创建循环依赖模块

让我们假设你有两个文件，a.py 和 b.py，他们之间相互引用，如下所示：

a.py:

    
    
    import b  
    def f():  
        return b.x    
    print f()

b.py:

    
    
    import a  
    x = 1 
    def g():  
        print a.f()

首先，让我们尝试引入 a.py：

>>> import a

1

可以正常工作。这也许是你感到很奇怪。毕竟，我们确实在这里引入了一个循环依赖的模块，我们推测这样会出问题的，不是吗？

答案就是在Python中，仅仅引入一个循环依赖的模块是没有问题的。如果一个模块已经被引入了，Python并不会去再次引入它。但是，根据每个模块要访问其他模块
中的函数和变量位置的不同，就很可能会遇到问题。

所以，回到我们这个例子，当我们引入 a.py 时，再引入 b.py 不会产生任何问题，因为当引入的时候，b.py 不需要 a.py 中定义任何东西。b.py
中唯一引用 a.py 中的东西是调用 a.f()。 但是那个调用是发生在g() 中的，并且 a.py 和 b.py 中都没有调用 g()。所以运行正常。

但是，如果我们尝试去引入b.py 会发生什么呢？（在这之前不引入a.py），如下所示:

    
    
    >>> import b

Traceback (most recent call last):

        File "<stdin>", line 1, in <module>

        File "b.py", line 1, in <module>

    import a

        File "a.py", line 6, in <module>

    print f()

        File "a.py", line 4, in f

    return b.x

AttributeError: 'module' object has no attribute 'x'

啊哦。 出问题了！此处的问题是，在引入b.py的过程中，Python尝试去引入 a.py，但是a.py 要调用f()，而f() 有尝试去访问
b.x。但是此时 b.x 还没有被定义呢。所以发生了 AttributeError 异常。

至少，解决这个问题很简单，只需修改b.py，使其在g()中引入 a.py：

    
    
    x = 1 
    def g():  
        import a    # 只有当g()被调用的时候才会引入a  
        print a.f()

现在，当我们再引入b，没有任何问题：

    
    
    >>> import b  
    >>> b.g()  
    1    # Printed a first time since module 'a' calls 'print f()' at the end  
    1    # Printed a second time, this one is our call to 'g'
    
    
    常见错误 8: 与Python标准库中的模块命名冲突

Python一个令人称赞的地方是它有丰富的模块可供我们“开箱即用”。但是，如果你没有有意识的注意的话，就很容易出现你写的模块和Python自带的标准库的模块
之间发生命名冲突的问题（如，你也许有一个叫 email.py 的模块，但这会和标准库中的同名模块冲突）。

这可能会导致很怪的问题，例如，你引入了另一个模块，但这个模块要引入一个Python标准库中的模块，由于你定义了一个同名的模块，就会使该模块错误的引入了你的模
块，而不是 stdlib 中的模块。这就会出问题了。

因此，我们必须要注意这个问题，以避免使用和Python标准库中相同的模块名。修改你包中的模块名要比通过 Python Enhancement
Proposal (PEP) 给Python提建议来修改标准库的模块名容易多了。

常见错误 #9: 未能解决Python 2和Python 3之间的差异

请看下面这个 filefoo.py:

    
    
    import sys  
    def bar(i):  
        if i == 1:  
            raise KeyError(1)  
        if i == 2:  
            raise ValueError(2)  
     
    def bad():  
        e = None 
        try:  
            bar(int(sys.argv[1]))  
        except KeyError as e:  
            print('key error')  
        except ValueError as e:  
            print('value error')  
        print(e)  
     
    bad()

在Python 2中运行正常：

$ python foo.py 1

key error

1

$ python foo.py 2

value error

2

但是，现在让我们把它在Python 3中运行一下：

$ python3 foo.py 1

key error

Traceback (most recent call last):

  File "foo.py", line 19, in <module>

    bad()

  File "foo.py", line 17, in bad

    print(e)

UnboundLocalError: local variable 'e' referenced before assignment

出什么问题了？ “问题”就是，在 Python 3 中，异常的对象在 except
代码块之外是不可见的。（这样做的原因是，它将保存一个对内存中堆栈帧的引用周期，直到垃圾回收器运行并且从内存中清除掉引用。了解更多技术细节请参考这里） 。

一种解决办法是在 except 代码块的外部作用域中定义一个对异常对象的引用，以便访问。下面的例子使用了该方法，因此最后的代码可以在Python 2 和
Python 3中运行良好。

    
    
    import sys  
    def bar(i):  
        if i == 1:  
            raise KeyError(1)  
        if i == 2:  
            raise ValueError(2)  
    def good():  
        exception = None 
        try:  
            bar(int(sys.argv[1]))  
        except KeyError as e:  
            exception = e  
            print('key error')  
        except ValueError as e:  
            exception = e  
            print('value error')  
        print(exception)  
     
    good()

在Py3k中运行:

$ python3 foo.py 1

key error

1

$ python3 foo.py 2

value error

2

正常!

(顺便提一下, 我们的 Python Hiring Guide 讨论了当我们把代码从Python 2 迁移到 Python
3时的其他一些需要知道的重要差异。)

### 常见错误 10: 误用__del__方法

假设你有一个名为 calledmod.py 的文件：

    
    
    import foo  
    class Bar(object):  
               ...  
        def __del__(self):  
            foo.cleanup(self.myhandle)

并且有一个名为 another_mod.py 的文件：

import mod

mybar = mod.Bar()

你会得到一个 AttributeError 的异常。

为什么呢？因为，正如这里所说，当解释器退出的时候，模块中的全局变量都被设置成了 None。所以，在上面这个例子中，当 __del__ 被调用时，foo
已经被设置成了None。

解决方法是使用 atexit.register() 代替。用这种方式，当你的程序结束执行时（意思是正常退出），你注册的处理程序会在解释器退出之前执行。

了解了这些，我们可以将上面 mod.py 的代码修改成下面的这样：

    
    
    import foo  
    import atexit  
    def cleanup(handle):  
        foo.cleanup(handle)  
    class Bar(object):  
        def __init__(self):  
            ...  
            atexit.register(cleanup, self.myhandle)

这种实现方式提供了一个整洁并且可信赖的方法用来在程序退出之前做一些清理工作。很显然，它是由foo.cleanup 来决定对绑定在 self.myhandle
上对象做些什么处理工作的，但是这就是你想要的。

总结

Python是一门强大的并且很灵活的语言，它有很多机制和语言规范来显著的提高你的生产力。和其他任何一门语言或软件一样，如果对它能力的了解有限，这很可能会给你
带来阻碍，而不是好处。正如一句谚语所说的那样 “knowing enough to be
dangerous”（译者注：意思是自以为已经了解足够了，可以做某事了，但其实不是）。

熟悉Python的一些关键的细微之处，像本文中所提到的那些（但不限于这些），可以帮助我们更好的去使用语言，从而避免一些常见的陷阱。

  

