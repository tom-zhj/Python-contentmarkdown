# 看完这篇，你就知道Python生成器是什么

生成器是 Python 初级开发者最难理解的概念之一，虽被认为是 Python
编程中的高级技能，但在各种项目中可以随处见到生成器的身影，你得不得去理解它、使用它、甚至爱上它。

  

提到生成器，总不可避免地要把迭代器拉出来对比着讲，生成器就是一个在行为上和迭代器非常类似的对象，如果把迭代器比作 Android 系统，那么生成器就是
iOS，二者功能上差不多，但是生成器更优雅。

  

### 什么是迭代器

  

顾名思义，迭代器就是用于迭代操作（for 循环）的对象，它像列表一样可以迭代获取其中的每一个元素，任何实现了 __next__ 方法 （python2 是
next）的对象都可以称为迭代器。

  

它与列表的区别在于，构建迭代器的时候，不像列表把所有元素一次性加载到内存，而是以一种延迟计算（lazy evaluation）方式返回元素，这正是它的优点。
比如列表含有中一千万个整数，需要占超过400M的内存，而迭代器只需要几十个字节的空间。因为它并没有把所有元素装载到内存中，而是等到调用 next
方法时候才返回该元素（按需调用 call by need 的方式，本质上 for 循环就是不断地调用迭代器的next方法）。

  

以斐波那契数列为例来实现一个迭代器：

    
    
    class Fib:
        def __init__(self, n):
            self.prev = 0
            self.cur = 1
            self.n = n
        def __iter__(self):
            return self
        def __next__(self):
            if self.n > 0:
                value = self.cur
                self.cur = self.cur + self.prev
                self.prev = value
                self.n -= 1
                return value
            else:
                raise StopIteration()
        # 兼容python2
        def __next__(self):
            return self.next()
    f = Fib(10)
    print([i for i in f])
    #[1, 1, 2, 3, 5, 8, 13, 21, 34, 55]

### 什么是生成器

  

知道迭代器之后，就可以正式进入生成器的话题了。普通函数用 return 返回一个值，和 Java 等其他语言是一样的，然而在 Python
中还有一种函数，用关键字 yield 来返回值，这种函数叫生成器函数，函数被调用时会返回一个生成器对象，生成器本质上还是一个迭代器，也是用在迭代操作中，因此
它有和迭代器一样的特性，唯一的区别在于实现方式上不一样，后者更加简洁

  

最简单的生成器函数：

    
    
    >>> def func(n):
    ...     yield n*2
    ...
    >>> func
    <function func at 0x00000000029F6EB8>
    >>> g = func(5)
    >>> g
    <generator object func at 0x0000000002908630>
    >>>

func 就是一个生成器函数，调用该函数时返回对象就是生成器 g ，这个生成器对象的行为和迭代器是非常相似的，可以用在 for 循环等场景中。注意
yield 对应的值在函数被调用时不会立刻返回，而是调用next方法时（本质上 for 循环也是调用 next 方法）才返回

    
    
    >>> g = func(5)
    >>> next(g)
    10
    >>> g = func(5)
    >>> for i in g:
    ...     print(i)
    ...
    10

那为什么要用生成器呢？显然，用生成器在逼格上要比迭代器高几个等级，它没有那么多冗长代码了，而且性能上一样的高效，为什么不用呢？来看看用生成器实现斐波那契数列
有多简单。

    
    
    def fib(n):
        prev, curr = 0, 1
        while n > 0:
            n -= 1
            yield curr
            prev, curr = curr, curr + prev
    print([i for i in fib(10)])
    #[1, 1, 2, 3, 5, 8, 13, 21, 34, 55]

  

### 生成器表达式

  

在前面一期「这样写代码更优雅」的文章里面曾经介绍过列表推导式（list
comprehension），生成器表达式与列表推导式长的非常像，但是它俩返回的对象不一样，前者返回生成器对象，后者返回列表对象。

    
    
    >>> g = (x*2 for x in range(10))
    >>> type(g)
    <type 'generator'>
    >>> l = [x*2 for x in range(10)]
    >>> type(l)
    <type 'list'>

前面已经介绍过生成器的优势，就是迭代海量数据时，显然生成器更合适。

  

