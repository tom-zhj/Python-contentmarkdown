# 详解python的super()的作用和原理

Python中对象方法的定义很怪异，第一个参数一般都命名为self（相当于其它语言的this），用于传递对象本身，而在调用的时候则不必显式传递，系统会自动传
递。

今天我们介绍的主角是super(), 在类的继承里面super()非常常用， 它解决了子类调用父类方法的一些问题， 父类多次被调用时只执行一次，
优化了执行逻辑，下面我们就来详细看一下。

  

举一个例子：

    
    
    
    class Foo:
      def bar(self, message):
        print(message)
    
    
    >>> Foo().bar("Hello, Python.")
    Hello, Python.

当存在继承关系的时候，有时候需要在子类中调用父类的方法，此时最简单的方法是把对象调用转换成类调用，需要注意的是这时self参数需要显式传递，例如：

    
    
    
    class FooParent:
      def bar(self, message):
        print(message)
    class FooChild(FooParent):
      def bar(self, message):
        FooParent.bar(self, message)
    
    
    >>> FooChild().bar("Hello, Python.")
    Hello, Python.

  

这样做有一些缺点，比如说如果修改了父类名称，那么在子类中会涉及多处修改，另外，Python是允许多继承的语言，如上所示的方法在多继承时就需要重复写多次，显得
累赘。为了解决这些问题，Python引入了super()机制，例子代码如下：

    
    
    
    class FooParent:
      def bar(self, message):
        print(message)
    class FooChild(FooParent):
      def bar(self, message):
        super(FooChild, self).bar(message)

  

    
    
    >>> FooChild().bar("Hello, Python.")
    Hello, Python.

表面上看 super(FooChild, self).bar(message)方法和FooParent.bar(self,
message)方法的结果是一致的，实际上这两种方法的内部处理机制大大不同，当涉及多继承情况时，就会表现出明显的差异来，直接给例子：

  

代码一：

    
    
    class A:
      def __init__(self):
        print("Enter A")
        print("Leave A")
    class B(A):
      def __init__(self):
        print("Enter B")
        A.__init__(self)
        print("Leave B")
    class C(A):
      def __init__(self):
        print("Enter C")
        A.__init__(self)
        print("Leave C")
    class D(A):
      def __init__(self):
        print("Enter D")
        A.__init__(self)
        print("Leave D")
    class E(B, C, D):
      def __init__(self):
        print("Enter E")
        B.__init__(self)
        C.__init__(self)
        D.__init__(self)
        print("Leave E")
    E()

  

结果：

  

Enter E

Enter B

Enter A

Leave A

Leave B

Enter C

Enter A

Leave A

Leave C

Enter D

Enter A

Leave A

Leave D

Leave E

  

执行顺序很好理解，唯一需要注意的是公共父类A被执行了多次。

  

代码二：

    
    
    class A:
      def __init__(self):
        print("Enter A")
        print("Leave A")
    class B(A):
      def __init__(self):
        print("Enter B")
        super(B, self).__init__()
        print("Leave B")
    class C(A):
      def __init__(self):
        print("Enter C")
        super(C, self).__init__()
        print("Leave C")
    class D(A):
      def __init__(self):
        print("Enter D")
        super(D, self).__init__()
        print("Leave D")
    class E(B, C, D):
      def __init__(self):
        print("Enter E")
        super(E, self).__init__()
        print("Leave E")
    E()

  

结果：

  

Enter E

Enter B

Enter C

Enter D

Enter A

Leave A

Leave D

Leave C

Leave B

Leave E

  

在super机制里可以保证公共父类仅被执行一次，至于执行的顺序，是按照MRO（Method Resolution Order）：方法解析顺序
进行的。后续会详细介绍一下这个MRO机制。

  

