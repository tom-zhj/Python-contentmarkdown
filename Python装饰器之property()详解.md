# Python装饰器之property()详解

### 1\. 何为装饰器？

官方定义：装饰器是一个很著名的设计模式，经常被用于有切面需求的场景，较为经典的有插入日志、性能测试、事务处理等。装饰器是解决这类问题的绝佳设计，有了装饰器，
我们就可以抽离出大量函数中与函数功能本身无关的雷同代码并继续重用。概括的讲，装饰器的作用就是为已经存在的对象添加额外的功能。

  

Python中总共包括三个内置装饰器：

  

① staticmethod

② classmethod

③ property

  

### 2\. 属性函数 property() 浅谈

  

#### 2.1 为什么要使用 property？

通常，我们在访问属性和给属性赋值的时候，都是对 类和实例 __dict__ 打交道的；但如果我们想要规范属性访问，有两种方式可用：①数据描述符 ，②.
property() 属性函数。

  

然而，我们知道，描述符相对比较复杂，对于新手来说，用起来很吃力，那么不妨试试property()，相对于描述符这个大的进程，property就相当于线程。

  

#### 2.2 函数原型：

property(fget=None, fset=None, fdel=None, doc=None)

#### 2.3 普通方法定义：

假设 calss Normal中有一个私有变量 __x，如下代码所示：

    
    
    #code 1
    class Normal:
        def __init__(self):
            self.__x = None
        def getx(self):
            return self.__x
        def setx(self, value):
            self.__x = value
        def delx(self):
            del self.__x
    tN = Normal()
    print(tN.__count)

输出结果（报错了）

    
    
    Traceback (most recent call last):
      File "C:/Users/Administrator/AppData/Local/Programs/Python/Python35/property.py", line 15, in <module>
        print(tN.__count)
    AttributeError: 'Normal' object has no attribute '__count'

  

为啥报错了呢？因为 实例tN的属性 __x 为私有属性，不能直接访问，为此我们只能调用内部定义的 方法；

    
    
    tN = Normal()
    tN.setx(10)
    print(tN.getx())

输出结果：

    
    
    6 10

  

使用内部的方法，可以容易的得到实例的或者类的私有属性值；

  

然而，如果我想把 class Normal 的 setx方法名改成了其它（如
Normal_setx）,外部很多地方用到了该函数，是不是我需要一个一个的去找该方法的调用地点，然后一个一个的改呢？

  

c语言或许会，但Python，一个高级语言，怎么会这么点事都解决不了呢？

那么，该如何解决以上问题呢？

  

其实有两种方法。

  

### 方法一：使用 属性函数property()

    
    
    class Normal:
        def __init__(self):
            self.__x = None
        def getx(self):
            print('getx(): self.__x=', self.__x)
            return self.__x
        def setx(self, value):
            self.__x = value
            print('setx()')
        def delx(self):
            print('delx()')
            del self.__x
        y = property(getx, setx, delx, "I'm a property")
    tN=Normal()
    tN.y=10
    tN.y
    del tN.y
    #输出结果：
    setx()
    getx(): self.__x= 10
    delx()

直接把方法当属性来操作了，非常方便

### 方法二：使用 @property 装饰器

    
    
    class Normal:
        
        def __init__(self):
            self.__x = None
        @property
        def xx(self):
            print('getx(): self.__x=', self.__x)
            return self.__x
        
        @xx.setter
        def xx(self, value):
            self.__x = value
            print('setx()')
        @xx.deleter
        def xx(self):
            print('delx()')
            del self.__x
    tN=Normal()
    tN.xx=10
    tN.xx
    del tN.xx
    #输出结果信息：
    setx()
    getx(): self.__x= 10
    delx()

跟方法一 输出同样的结果，证明，这两种方法都可行的（注意：第一个一定是 @property(替代getter哦，不然会报错））。

  

