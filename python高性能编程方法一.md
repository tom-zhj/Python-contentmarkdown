# python高性能编程方法一

阅读 Zen of Python,在Python解析器中输入 import this. 一个犀利的Python新手可能会注意到"解析"一词,
认为Python不过是另一门脚本语言. "它肯定很慢!"

毫无疑问Python程序没有编译型语言高效快速. 甚至Python拥护者们会告诉你Python不适合这些领域.
然而,YouTube已用Python服务于每小时4千万视频的请求. 你所要做的就是编写高效的代码和需要时使用外部实现(C/C++)代码.
这里有一些建议,可以帮助你成为一个更好的Python开发者:

1\. 使用内建函数:    你可以用Python写出高效的代码,但很难击败内建函数. 经查证. 他们非常快速.2.使用join()连接字符串.
你可以使用 "+" 来连接字符串. 但由于string在Python中是不可变的,每一个"+"操作都会创建一个新的字符串并复制旧内容.
常见用法是使用Python的数组模块单个的修改字符;当完成的时候,使用 join() 函数创建最终字符串.

     >>> #This is good to glue a large number of strings

     >>> for chunk in input():

     >>>    my_string.join(chunk)

3\. 使用Python多重赋值，交换变量

   这在Python中即优雅又快速:

     >>> x, y = y, x

     这样很慢:

     >>> temp = x

     >>> x = y

     >>> y = temp

4\. 尽量使用局部变量

   Python 检索局部变量比检索全局变量快. 这意味着,避免 "global" 关键字.

5\. 尽量使用 "in"

     使用 "in" 关键字. 简洁而快速.

     >>> for key in sequence:

     >>>     print “found”

6\. 使用延迟加载加速

     將 "import" 声明移入函数中,仅在需要的时候导入. 换句话说,如果某些模块不需马上使用,稍后导入他们.
例如,你不必在一开使就导入大量模块而加速程序启动. 该技术不能提高整体性能. 但它可以帮助你更均衡的分配模块的加载时间.

7\. 为无限循环使用 "while 1"

   有时候在程序中你需一个无限循环.(例如一个监听套接字的实例) 尽管 "while true" 能完成同样的事, 但 "while 1" 是单步运算.
这招能提高你的Python性能.

      >>> while 1:

     >>>    #do stuff, faster with while 1

     >>> while True:

     >>>    # do stuff, slower with wile True

8\. 使用list comprehension

   从Python 2.0 开始,你可以使用 list comprehension 取代大量的 "for" 和 "while" 块. 使用List
comprehension通常更快，Python解析器能在循环中发现它是一个可预测的模式而被优化.额外好处是，list
comprehension更具可读性（函数式编程），并在大多数情况下，它可以节省一个额外的计数变量。例如，让我们计算1到10之间的偶数个数：

     >>> # the good way to iterate a range

     >>> evens = [ i for i in range(10) if i%2 == 0]

     >>> [0, 2, 4, 6, 8]

     >>> # the following is not so Pythonic

     >>> i = 0

     >>> evens = []

     >>> while i < 10:

     >>>    if i %2 == 0: evens.append(i)

     >>>    i += 1

     >>> [0, 2, 4, 6, 8]

9\. 使用xrange()处理长序列：

   这样可为你节省大量的系统内存，因为xrange()在序列中每次调用只产生一个整数元素。而相反
range()，它將直接给你一个完整的元素列表，用于循环时会有不必要的开销。

10\. 使用 Python generator：

     这也可以节省内存和提高性能。例如一个视频流，你可以一个一个字节块的发送，而不是整个流。例如，

     >>> chunk = ( 1000 * i for i in xrange(1000))

     >>> chunk

     <generator object  at 0x7f65d90dcaa0>

     >>> chunk.next()

     0

     >>> chunk.next()

     1000

     >>> chunk.next()

     2000

11\. 了解itertools模块：

     该模块对迭代和组合是非常有效的。让我们生成一个列表[1，2，3]的所有排列组合,仅需三行Python代码：

     >>> import itertools

     >>> iter = itertools.permutations([1,2,3])

     >>> list(iter)

     [(1, 2, 3), (1, 3, 2), (2, 1, 3), (2, 3, 1), (3, 1, 2), (3, 2, 1)]

12\. 学习bisect模块保持列表排序：

     这是一个免费的二分查找实现和快速插入有序序列的工具。也就是说，你可以使用：

     >>> import bisect

     >>> bisect.insort(list, element)

     你已將一个元素插入列表中, 而你不需要再次调用 sort() 来保持容器的排序, 因为这在长序列中这会非常昂贵.

13\. 理解Python列表，实际上是一个数组：

     Python中的列表实现并不是以人们通常谈论的计算机科学中的普通单链表实现的。Python中的列表是一个数组。也就是说，你可以以常量时间O(1)
检索列表的某个元素，而不需要从头开始搜索。这有什么意义呢？ Python开发人员使用列表对象insert（）时, 需三思. 例如：>>>
list.insert（0，item）

     在列表的前面插入一个元素效率不高, 因为列表中的所有后续下标不得不改变. 然而，您可以使用list.append()在列表的尾端有效添加元素.
挑先deque，如果你想快速的在两插入或时。它是快速的，因为在Python中的deque用双链表实现。不再多说。

14\. 使用dict 和 set 测试成员：      检查一个元素是在dicitonary或set是否存在
这在Python中非常快的。这是因为dict和set使用哈希表来实现。查找效率可以达到O(1)。因此，如果您需要经常检查成员，使用 set 或
dict做为你的容器.

     >>> mylist = ['a', 'b', 'c'] #Slower, check membership with list:

     >>> ‘c’ in mylist

     >>> True

     >>> myset = set(['a', 'b', 'c']) # Faster, check membership with set:

     >>> ‘c’ in myset:

     >>> True

15\. 使用Schwartzian Transform 的 sort():

     原生的list.sort（）函数是非常快的。
Python会按自然顺序排序列表。有时，你需要非自然顺序的排序。例如，你要根据服务器位置排序的IP地址。
Python支持自定义的比较，你可以使用list.sort（CMP（）），这会比list.sort（）慢，因为增加了函数调用的开销。如果性能有问 题
，你可以申请Guttman-Rosler Transform,基于Schwartzian Transform.
它只对实际的要用的算法有兴趣，它的简要工作原理是，你可以变换列表，并调用Python内置list.sort（） - >
更快，而无需使用list.sort（CMP（） ）->慢。

16\. Python装饰器缓存结果：

     “@”符号是Python的装饰语法。它不只用于追查，锁或日志。你可以装饰一个Python函数，记住调用结果供后续使用。这种技术被称为memoiza
tion的。下面是一个例子：

     >>> from functools import wraps

     >>> def memo(f):

     >>>    cache = { }

     >>>    @wraps(f)

     >>>    def  wrap(*arg):

     >>>        if arg not in cache: cache['arg'] = f(*arg)

     >>>        return cache['arg']

     >>>    return wrap

     我们也可以对 Fibonacci 函数使用装饰器:

     >>> @memo

     >>> def fib(i):

     >>>    if i < 2: return 1

     >>>    return fib(i-1) + fib(i-2)



    这里的关键思想是:增强函数(装饰)函数,记住每个已经计算的Fibonacci值;如果它们在缓存中,就不需要再计算了.

17\. 理解Python的GIL（全局解释器锁）：

     GIL是必要的，因为CPython的内存管理是非线程安全的。你不能简单地创建多个线程，并希望Python能在多核心的机器上运行得更快。这是因为
GIL將会防止多个原生线程同时执行Python字节码。换句话说，GIL將序列化您的所有线程。然而，您可以使用线程管理多个派生进程加速程序，这些程
序独立的运行于你的Python代码外。

18\. 像熟悉文档一样的熟悉Python源代码：

     Python有些模块为了性能使用C实现。当性能至关重要而官方文档不足时，可以自由探索源代码。你可以找到底层的数据结构和算法。
Python的源码库就是一个很棒的地方：http://svn.python.org/view/python/trunk/Modules

结论：

这些不能替代大脑思考. 打开引擎盖充分了解是开发者的职责,使得他们不会快速拼凑出一个垃圾设计. 本文的Python建议可以帮助你获得好的性能.
如果速度还不够快, Python將需要借助外力:分析和运行外部代码.我们將在本文的第二部分中涉及.

  

