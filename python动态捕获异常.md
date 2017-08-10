# python动态捕获异常

在讨论动态捕获异常时让我大吃一惊的是,可以让我找到隐藏的Bug和乐趣...

有问题的代码

  

下面的代码来自一个产品中看起来是好的抽象代码 - slightly(!) .这是调用一些统计数据的函数,然后进行处理 .
首先是用socket连接获取一个值,可能发生了socket错误.由于统计数据在系统中不是至关重要的,我们只是记一下日志错误并继续往下走.

(请注意,这篇文章我使用doctest测试的 - 这代表代码可以运行!)

>>> def get_stats():

...     pass

...

>>> def do_something_with_stats(stats):

...     pass

...

>>> try:

...     stats = get_stats()

... except socket.error:

...     logging.warning("Can't get statistics")

... else:

...     do_something_with_stats(stats)

查找

  

我们测试时并没有发现不妥, 但实际上我们注意到静态分析报告显示一个问题:

$ flake8 filename.py

filename.py:351:1: F821 undefined name 'socket'

filename.py:352:1: F821 undefined name 'logging'

显然是我们没测试,这个问题是代码中我们没有引用socket 和 logging
两个模块.使我感到惊奇的是,这并没有预先抛出NameError错,我以为它会查找这些异常语句中的一些名词,如它需要捕捉这些异常,它需要知道些什么呢!

事实证明并非如此,异常语句的查找是延迟完成的,只是评估时抛出异常. 不只是名称延迟查找,也可以定制显示声明异常做为'参数(argument)'.

这可能是好事,坏事,或者是令人厌恶的.

好事(上段中提到的)

异常参数可以以任意形式数值传递. 这样就允许了异常的动态参数被捕获.

>>> def do_something():

...    blob

...

>>> def attempt(action, ignore_spec):

...     try:

...         action()

...     except ignore_spec:

...         pass

...

>>> attempt(do_something, ignore_spec=(NameError, TypeError))

>>> attempt(do_something, ignore_spec=TypeError)

Traceback (most recent call last):

 ...

NameError: global name 'blob' is not defined

坏事(上段中提到的)

这种明显的弊端就是异常参数中的错误通常只有在异常触发之后才会被注意到,不过为时已晚.当用异常去捕获不常见的事件时(例如:以写方式打开文件失败),除非做个一个
特定的测试用例,否则只有当一个异常(或者任何异常)被触发的时候才会知道, 届时记录下来并且查看是否有匹配的异常, 并且抛出它自己的错误异常 -
这是一个NameError通常所做的事情.

>>> def do_something():

...     return 1, 2

...

>>> try:

...     a, b = do_something()

... except ValuError:  # oops - someone can't type

...     print("Oops")

... else:

...     print("OK!")   # we are 'ok' until do_something returns a triple...

OK!

令人讨厌的(上段中提到的)

>>> try:

...    TypeError = ZeroDivisionError  # now why would we do this...?!

...    1 / 0

... except TypeError:

...    print("Caught!")

... else:

...    print("ok")

...

Caught!

不仅仅是异常参数通过名称查找, - 其它的表达式也是这样工作的:

>>> try:

...     1 / 0

... except eval(''.join('Zero Division Error'.split())):

...     print("Caught!")

... else:

...     print("ok")

...

Caught!

异常参数不仅仅只能在运行时确定,它甚至可以使用在生命周期内的异常的信息. 以下是一个比较费解的方式来捕捉抛出的异常 - 但也只能如此了:

>>> import sys

>>> def current_exc_type():

...     return sys.exc_info()[0]

...

>>> try:

...     blob

... except current_exc_type():

...     print ("Got you!")

...

Got you!

很明显这才是我们真正要寻找的当我们写异常处理程序时, 我们应该首先想到的就是这种

（字节）代码

为了确认它是如何在异常处理工作中出现的,我在一个异常的例子中运行 dis.dis(). (注意 这里的分解是在Python2.7 下 -
不同的字节码是Python 3.3下产生的,但这基本上是类似的):

>>> import dis

>>> def x():

...     try:

...         pass

...     except Blobbity:

...         print("bad")

...     else:

...         print("good")

...

>>> dis.dis(x)  # doctest: +NORMALIZE_WHITESPACE

 2           0 SETUP_EXCEPT             4 (to 7)

<BLANKLINE>

 3           3 POP_BLOCK

             4 JUMP_FORWARD            22 (to 29)

<BLANKLINE>

 4     >>    7 DUP_TOP

             8 LOAD_GLOBAL              0 (Blobbity)

            11 COMPARE_OP              10 (exception match)

            14 POP_JUMP_IF_FALSE       28

            17 POP_TOP

            18 POP_TOP

            19 POP_TOP

<BLANKLINE>

 5          20 LOAD_CONST               1 ('bad')

            23 PRINT_ITEM

            24 PRINT_NEWLINE

            25 JUMP_FORWARD             6 (to 34)

       >>   28 END_FINALLY

<BLANKLINE>

 7     >>   29 LOAD_CONST               2 ('good')

            32 PRINT_ITEM

            33 PRINT_NEWLINE

       >>   34 LOAD_CONST               0 (None)

            37 RETURN_VALUE

这显示出了我原来预期的问题(issue). 异常处理"看起来"完全是按照Python内部机制在运行. 这一步完全没有必要知道关于后续的异常“捕获”语句,
并且如果没有异常抛出它们将被完全忽略了.SETUP_EXCEPT并不关心发生了什么, 仅仅是如果发生了异常,
第一个处理程序应该被评估,然后第二个,以此类推.

每个处理程序都有两部分组成: 获得一个异常的规则, 和刚刚抛出的异常进行对比. 一切都是延迟的, 一切看起来正如对你的逐行的代码的预期一样,
从解释器的角度来考虑. 没有任务聪明的事情发生了,只是突然使得它看起来非常聪明.

总结

虽然这种动态的异常参数让我大吃一惊, 但是这当中包含很多有趣的应用. 当然去实现它们当中的许多或许是个馊主意,呵呵

有时并不能总是凭直觉来确认有多少Python特性的支持 - 例如 在类作用域内 表达式和声明都是被显式接受的, (而不是函数, 方法,
全局作用域),但是并不是所有的都是如此灵活的. 虽然(我认为)那将是十分美好的, 表达式被禁止应用于装饰器 - 以下是Python语法错误:

@(lambda fn: fn)

def x():

  pass

这个是尝试动态异常参数通过给定类型传递给第一个异常的例子, 静静的忍受重复的异常:

>>> class Pushover(object):

...     exc_spec = set()

...

...     def attempt(self, action):

...         try:

...             return action()

...         except tuple(self.exc_spec):

...             pass

...         except BaseException as e:

...             self.exc_spec.add(e.__class__)

...             raise

...

>>> pushover = Pushover()

>>>

>>> for _ in range(4):

...     try:

...         pushover.attempt(lambda: 1 / 0)

...     except:

...         print ("Boo")

...     else:

...         print ("Yay!")

Boo

Yay!

Yay!

Yay!

  

