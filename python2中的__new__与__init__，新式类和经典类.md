# python2中的__new__与__init__，新式类和经典类

在python2.x中，从object继承得来的类称为新式类（如class A(object)）不从object继承得来的类称为经典类（如class
A()）

新式类跟经典类的差别主要是以下几点:

　　1. 新式类对象可以直接通过__class__属性获取自身类型:type

　　2. 继承搜索的顺序发生了改变,经典类多继承时属性搜索顺序: 先深入继承树左侧，再返回，开始找右侧（即深度优先搜索）;新式类多继承属性搜索顺序: 先水平搜索，然后再向上移动

例子：

经典类: 搜索顺序是(D,B,A,C)

    
    
    >>> class A: attr = 1
    ...
    >>> class B(A): pass
    ...
    >>> class C(A): attr = 2
    ...
    >>> class D(B,C): pass
    ...
    >>> x = D()
    >>> x.attr
    1

新式类继承搜索程序是宽度优先

新式类：搜索顺序是(D,B,C,A)

    
    
    >>> class A(object): attr = 1
    ...
    >>> class B(A): pass
    ...
    >>> class C(A): attr = 2
    ...
    >>> class D(B,C): pass
    ...
    >>> x = D()
    >>> x.attr
    2

　　3. 新式类增加了__slots__内置属性, 可以把实例属性的种类锁定到__slots__规定的范围之中。

　　4. 新式类增加了__getattribute__方法

　　5.新式类内置有__new__方法而经典类没有__new__方法而只有__init__方法

注意：Python 2.x中默认都是经典类，只有显式继承了object才是新式类

　　   而Python 3.x中默认都是新式类（也即object类默认是所有类的祖先），不必显式的继承object（可以按照经典类的定义方式写一个经典类并
分别在python2.x和3.x版本中使用dir函数检验下。

例如：class A()：

　　　　　　pass

　　　 print(dir(A))

会发现在2.x下没有__new__方法而3.x下有。

接下来说下__new__方法和__init__的区别：

在python中创建类的一个实例时，如果该类具有__new__方法，会先调用__new__方法，__new__方法接受当前正在实例化的类作为第一个参数（这个
参数的类型是type，这个类型在c和python的交互编程中具有重要的角色，感兴趣的可以搜下相关的资料），其返回值是本次创建产生的实例，也就是我们熟知的__
init__方法中的第一个参数self。那么就会有一个问题，这个实例怎么得到？

注意到有__new__方法的都是object类的后代，因此如果我们自己想要改写__new__方法（注意不改写时在创建实例的时候使用的是父类的__new__方
法，如果父类没有则继续上溯）可以通过调用object的__new__方法类得到这个实例（这实际上也和python中的默认机制基本一致），如：

    
    
    class display(object):
        def __init__(self, *args, **kwargs):
            print("init")
        def __new__(cls, *args, **kwargs):
            print("new")
            print(type(cls))
            return object.__new__(cls, *args, **kwargs)   
    a=display()

运行上述代码会得到如下输出：

new

<class 'type'>

init

因此我们可以得到如下结论：

在实例创建过程中__new__方法先于__init__方法被调用，它的第一个参数类型为type。

如果不需要其它特殊的处理，可以使用object的__new__方法来得到创建的实例（也即self)。

于是我们可以发现，实际上可以使用其它类的__new__方法类得到这个实例，只要那个类或其父类或祖先有__new__方法。

    
    
    class another(object):
        def __new__(cls,*args,**kwargs):
            print("newano")
            return object.__new__(cls, *args, **kwargs)   
    class display(object):
        def __init__(self, *args, **kwargs):
            print("init")
        def __new__(cls, *args, **kwargs):
            print("newdis")
            print(type(cls))
            return another.__new__(cls, *args, **kwargs)   
    a=display()

上面的输出是：

    
    
    newdis
    <class 'type'>
    newano
    init

所有我们发现__new__和__init__就像这么一个关系，__init__提供生产的原料self(但并不保证这个原料来源正宗，像上面那样它用的是另一个不
相关的类的__new__方法类得到这个实例)，而__init__就用__new__给的原料来完善这个对象（尽管它不知道这些原料是不是正宗的）

  

