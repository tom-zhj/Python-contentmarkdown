# 利用Python延迟初始化提升性能

所谓类属性的延迟计算就是将类的属性定义成一个property，只在访问的时候才会计算，而且一旦被访问后，结果将会被缓存起来，不用每次都计算。构造一个延迟计算
属性的主要目的是为了提升性能

### property

在切入正题之前，我们了解下property的用法，property可以将属性的访问转变成方法的调用。

    
    
    class Circle(object): 
      def __init__(self, radius): 
        self.radius = radius 
      
      @property
      def area(self): 
        return 3.14 * self.radius ** 2
      
    c = Circle(4) 
    print c.radius 
    print c.area

可以看到，area虽然是定义成一个方法的形式，但是加上@property后，可以直接执行c.area，当成属性访问。

  

现在问题来了，每次调用c.area，都会计算一次，太浪费cpu了，怎样才能只计算一次呢?这就是lazy property

  

### 代码实现

    
    
    class LazyProperty(object):
      def __init__(self, func):
        self.func = func
      def __get__(self, instance, owner):
        if instance is None:
          return self
        else:
          value = self.func(instance)
          setattr(instance, self.func.__name__, value)
          return value
    import math
    class Circle(object):
      def __init__(self, radius):
        self.radius = radius
      @LazyProperty
      def area(self):
        print 'Computing area'
        return math.pi * self.radius ** 2
      @LazyProperty
      def perimeter(self):
        print 'Computing perimeter'
        return 2 * math.pi * self.radius

### 说明

定义了一个延迟计算的装饰器类LazyProperty。Circle是用于测试的类，Circle类有是三个属性半径(radius)、面积(area)、周长(p
erimeter)。面积和周长的属性被LazyProperty装饰，下面来试试LazyProperty的魔法：

    
    
    >>> c = Circle(2)
    >>> print c.area
    Computing area
    12.5663706144
    >>> print c.area
    12.5663706144

在area()中每计算一次就会打印一次“Computing area”，而连续调用两次c.area后“Computing
area”只被打印了一次。这得益于LazyProperty,只要调用一次后，无论后续调用多少次都不会重复计算。

  

