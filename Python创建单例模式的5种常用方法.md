# Python创建单例模式的5种常用方法

所谓单例，是指一个类的实例从始至终只能被创建一次。

### 方法1

如果想使得某个类从始至终最多只有一个实例，使用__new__方法会很简单。Python中类是通过__new__来创建实例的：

    
    
    class Singleton(object):
        def __new__(cls,*args,**kwargs):
            if not hasattr(cls,'_inst'):
                cls._inst=super(Singleton,cls).__new__(cls,*args,**kwargs)
            return cls._inst
    if __name__=='__main__':
        class A(Singleton):
            def __init__(self,s):
                self.s=s      
        a=A('apple')   
        b=A('banana')
        print id(a),a.s
        print id(b),b.s

结果：

29922256 banana

29922256 banana

通过__new__方法，将类的实例在创建的时候绑定到类属性_inst上。如果cls._inst为None，说明类还未实例化，实例化并将实例绑定到cls._i
nst，以后每次实例化的时候都返回第一次实例化创建的实例。注意从Singleton派生子类的时候，不要重载__new__。

### 方法2：

有时候我们并不关心生成的实例是否具有同一id，而只关心其状态和行为方式。我们可以允许许多个实例被创建，但所有的实例都共享状态和行为方式：

    
    
    class Borg(object):
        _shared_state={}
        def __new__(cls,*args,**kwargs):
            obj=super(Borg,cls).__new__(cls,*args,**kwargs)
            obj.__dict__=cls._shared_state
            return obj

将所有实例的__dict__指向同一个字典，这样实例就共享相同的方法和属性。对任何实例的名字属性的设置，无论是在__init__中修改还是直接修改，所有的实
例都会受到影响。不过实例的id是不同的。要保证类实例能共享属性，但不和子类共享，注意使用cls._shared_state,而不是Borg._shared_
state。

因为实例是不同的id，所以每个实例都可以做字典的key：

    
    
    if __name__=='__main__':
        class Example(Borg):
            pass
        a=Example()
        b=Example()
        c=Example()
        adict={}
        j=0
        for i in a,b,c:
            adict[i]=j
            j+=1
        for i in a,b,c:
            print adict[i]

结果：

0

1

2

如果这种行为不是你想要的，可以为Borg类添加__eq__和__hash__方法，使其更接近于单例模式的行为：

    
    
    class Borg(object):
        _shared_state={}
        def __new__(cls,*args,**kwargs):
            obj=super(Borg,cls).__new__(cls,*args,**kwargs)
            obj.__dict__=cls._shared_state
            return obj
        def __hash__(self):
            return 1
        def __eq__(self,other):
            try:
                return self.__dict__ is other.__dict__
            except:
                return False
    if __name__=='__main__':
        class Example(Borg):
            pass
        a=Example()
        b=Example()
        c=Example()
        adict={}
        j=0
        for i in a,b,c:
            adict[i]=j
            j+=1
        for i in a,b,c:
            print adict[i]

结果：

2

2

2

所有的实例都能当一个key使用了。

###  方法3

当你编写一个类的时候，某种机制会使用类名字，基类元组，类字典来创建一个类对象。新型类中这种机制默认为type，而且这种机制是可编程的，称为元类__metac
lass__ 。

    
    
    class Singleton(type):
        def __init__(self,name,bases,class_dict):
            super(Singleton,self).__init__(name,bases,class_dict)
            self._instance=None
        def __call__(self,*args,**kwargs):
            if self._instance is None:
                self._instance=super(Singleton,self).__call__(*args,**kwargs)
            return self._instance
    if __name__=='__main__':
        class A(object):
            __metaclass__=Singleton       
        a=A()
        b=A()
        print id(a),id(b)

结果：

34248016 34248016

id是相同的。

例子中我们构造了一个Singleton元类，并使用__call__方法使其能够模拟函数的行为。构造类A时，将其元类设为Singleton，那么创建类对象A时
，行为发生如下：

A=Singleton(name,bases,class_dict),A其实为Singleton类的一个实例。

创建A的实例时，A()=Singleton(name,bases,class_dict)()=Singleton(name,bases,class_dict
).__call__()，这样就将A的所有实例都指向了A的属性_instance上，这种方法与方法1其实是相同的。

###  方法4

python中的模块module在程序中只被加载一次，本身就是单例的。可以直接写一个模块，将你需要的方法和属性，写在模块中当做函数和模块作用域的全局变量即可
，根本不需要写类。

而且还有一些综合模块和类的优点的方法：

    
    
    class _singleton(object):
        class ConstError(TypeError):
            pass
        def __setattr__(self,name,value):
            if name in self.__dict__:
                raise self.ConstError
            self.__dict__[name]=value
        def __delattr__(self,name):
            if name in self.__dict__:
                raise self.ConstError
            raise NameError
    import sys
    sys.modules[__name__]=_singleton()

python并不会对sys.modules进行检查以确保他们是模块对象，我们利用这一点将模块绑定向一个类对象，而且以后都会绑定向同一个对象了。

将代码存放在single.py中：

    
    
    >>> import single
    >>> single.a=1
    >>> single.a=2

ConstError

>>> del single.a

ConstError

### 方法5

最简单的方法：

    
    
    class singleton(object):
        pass
    singleton=singleton()

将名字singleton绑定到实例上，singleton就是它自己类的唯一对象了。

  

