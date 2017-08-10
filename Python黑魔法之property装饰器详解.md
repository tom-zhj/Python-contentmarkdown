# Python黑魔法之property装饰器详解

@property装饰器能把一个方法变成属性一样来调用,下面我们就一起来看看Python黑魔法@property装饰器的使用技巧解析

@property有什么用呢?表面看来,就是将一个方法用属性的方式来访问.

上代码,代码最清晰了.

    
    
    class Circle(object): 
      def __init__(self, radius): 
        self.radius = radius 
     
      @property 
      def area(self): 
        return 3.14 * self.radius ** 2 
     
    c = Circle(4) 
    print c.radius 
    print c.area

  

可以看到,area虽然是定义成一个方法的形式,但是加上@property后,可以直接c.area,当成属性访问.

现在问题来了,每次调用c.area,都会计算一次,太浪费cpu了,怎样才能只计算一次呢?这就是lazy property.

    
    
    class lazy(object): 
      def __init__(self, func): 
        self.func = func 
     
      def __get__(self, instance, cls): 
        val = self.func(instance) 
        setattr(instance, self.func.__name__, val) 
        return val 
     
    class Circle(object): 
      def __init__(self, radius): 
        self.radius = radius 
     
      @lazy 
      def area(self): 
        print 'evalute' 
        return 3.14 * self.radius ** 2 
     
    c = Circle(4) 
    print c.radius 
    print c.area 
    print c.area 
    print c.area

可以看到,'evalute'只输出了一次,对@lazy的机制应该很好理解.

在这里,lazy类有__get__方法,说明是个描述器,第一次执行c.area的时候,因为顺序问题,先去c.__dict__中找,没找到,就去类空间找,在类
Circle中,有area()方法,于是就被__get__拦截.

在__get__中,调用实例的area()方法算出结果,并动态给实例添加个同名属性把结果赋给它,即加到c.__dict__中去.

再次执行c.area的时候,先去c.__dict__找,因为此时已经有了,就不会经过area()方法和__get__了.

注意点

请注意以下代码场景：

代码片段1：

    
    
    class Parrot(object): 
      def __init__(self): 
        self._voltage = 100000 
     
      @property 
      def voltage(self): 
        """Get the current voltage.""" 
        return self._voltage 
     
    if __name__ == "__main__": 
      # instance 
      p = Parrot() 
      # similarly invoke "getter" via @property 
      print p.voltage 
      # update, similarly invoke "setter" 
      p.voltage = 12

代码片段2：

    
    
    class Parrot: 
      def __init__(self): 
        self._voltage = 100000 
     
      @property 
      def voltage(self): 
        """Get the current voltage.""" 
        return self._voltage 
     
    if __name__ == "__main__": 
      # instance 
      p = Parrot() 
      # similarly invoke "getter" via @property 
      print p.voltage 
      # update, similarly invoke "setter" 
      p.voltage = 12

代码1、2的区别在于

class Parrot(object):

在python2下，分别运行测试

片段1：将会提示一个预期的错误信息 AttributeError: can't set attribute

片段2：正确运行

参考python2文档，@property将提供一个ready-only
property，以上代码没有提供对应的@voltage.setter，按理说片段2代码将提示运行错误，在python2文档中，我们可以找到以下信息：

BIF:

property([fget[, fset[, fdel[, doc]]]])

Return a property attribute for new-style classes (classes that derive from
object).

原来在python2下，内置类型 object
并不是默认的基类，如果在定义类时，没有明确说明的话（代码片段2），我们定义的Parrot（代码片段2）将不会继承object

而object类正好提供了我们需要的@property功能，在文档中我们可以查到如下信息：

new-style class

Any class which inherits from object. This includes all built-in types like
list and dict. Only new-style classes can use Python's newer, versatile
features like __slots__, descriptors, properties, and __getattribute__().

  

同时我们也可以通过以下方法来验证

    
    
    class A: 
      pass 
    >>type(A) 
    <type 'classobj'>

  

    
    
    class A(object): 
      pass 
    >>type(A) 
    <type 'type'>

从返回的<type 'classobj'>，<type 'type'>可以看出<type 'type'>是我们需要的object类型（python 3.0
将object类作为默认基类，所以都将返回<type 'type'>）

为了考虑代码的python 版本过渡期的兼容性问题，我觉得应该定义class文件的时候，都应该显式定义object，做为一个好习惯

最后的代码将如下：

    
    
    class Parrot(object): 
      def __init__(self): 
        self._voltage = 100000 
      @property 
      def voltage(self): 
        """Get the current voltage.""" 
        return self._voltage 
      @voltage.setter 
      def voltage(self, new_value): 
        self._voltage = new_value 
     
    if __name__ == "__main__": 
      # instance 
      p = Parrot() 
      # similarly invoke "getter" via @property 
      print p.voltage 
      # update, similarly invoke "setter" 
      p.voltage = 12

  

另外，@property是在2.6、3.0新增的，2.5没有该功能。

  

