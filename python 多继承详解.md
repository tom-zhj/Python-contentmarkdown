# python 多继承详解


    class A(object):    # A must be new-style class
       def __init__(self):
        print "enter A"
        print "leave A"
     
    class B(C):     # A --> C
       def __init__(self):
        print "enter B"
        super(B, self).__init__()
        print "leave B"
    
    
    在我们的印象中，对于super(B, self).__init__()是这样理解的：super(B, self)首先找到B的父类（就是类A），然后把类B的对象self转换为类A的对象，然后“被转换”的类A对象调用自己的__init__函数。  
      
    有一天某同事设计了一个相对复杂的类体系结构（我们先不要管这个类体系设计得是否合理，仅把这个例子作为一个题目来研究就好），代码如下  
    
    
    代码段4：
    
    
    class A(object):
        def __init__(self):
            print "enter A"
            print "leave A"
     
    class B(object):
        def __init__(self):
            print "enter B"
            print "leave B"
     
    class C(A):
        def __init__(self):
            print "enter C"
            super(C, self).__init__()
            print "leave C"
     
    class D(A):
        def __init__(self):
            print "enter D"
            super(D, self).__init__()
            print "leave D"
            class E(B, C):
            def __init__(self):
            print "enter E"
            B.__init__(self)
            C.__init__(self)
            print "leave E"
     
    class F(E, D):
        def __init__(self):
            print "enter F"
            E.__init__(self)
            D.__init__(self)
            print "leave F"

f = F() ，结果如下：

    
    
    enter F enter E enter B leave B enter C enter D enter A leave A leave D leave C leave E enter D enter A leave A leave D leave F  
      
    

   明显地，类A和类D的初始化函数被重复调用了2次，这并不是我们所期望的结果！我们所期望的结果是最多只有类A的初始化函数被调用2次——其实这是多继承的类体
系必须面对的问题。我们把代码段4的类体系画出来，如下图：

   object  
  |       \  
  |        A  
  |      / |  
  B  C  D  
   \   /   |  
     E    |  
       \   |  
         F

   按我们对super的理解，从图中可以看出，在调用类C的初始化函数时，应该是调用类A的初始化函数，但事实上却调用了类D的初始化函数。好一个诡异的问题！

也就是说，mro中记录了一个类的所有基类的类类型序列。查看mro的记录，发觉包含7个元素，7个类名分别为：

F E B C D A object

　　从而说明了为什么在C.__init__中使用super(C, self).__init__()会调用类D的初始化函数了。 ???

　　我们把代码段4改写为：

    
    
    代码段5：
    
    
    class A(object):
        def __init__(self):
            print "enter A"
            super(A, self).__init__()  # new
            print "leave A"
     
    class B(object):
        def __init__(self):
            print "enter B"
            super(B, self).__init__()  # new
            print "leave B"
     
    class C(A):
        def __init__(self):
            print "enter C"
            super(C, self).__init__()
            print "leave C"
     
    class D(A):
        def __init__(self):
            print "enter D"
            super(D, self).__init__()
            print "leave D"
            class E(B, C):
            def __init__(self):
            print "enter E"
            super(E, self).__init__()  # change
            print "leave E"
     
    class F(E, D):
        def __init__(self):
            print "enter F"
            super(F, self).__init__()  # change
            print "leave F"

f = F()，执行结果：

    
    
    enter F enter E enter B enter C enter D enter A leave A leave D leave C leave B leave E leave F  
      
    可见，F的初始化不仅完成了所有的父类的调用，而且保证了每一个父类的初始化函数只调用一次。  
    

小结

　　1. super并不是一个函数，是一个类名，形如super(B, self)事实上调用了super类的初始化函数，  
      产生了一个super对象；  
　　2. super类的初始化函数并没有做什么特殊的操作，只是简单记录了类类型和具体实例；  
　　3. super(B, self).func的调用并不是用于调用当前类的父类的func函数；  
　　4. Python的多继承类是通过mro的方式来保证各个父类的函数被逐一调用，而且保证每个父类函数  
      只调用一次（如果每个类都使用super）；  
　　5. 混用super类和非绑定的函数是一个危险行为，这可能导致应该调用的父类函数没有调用或者一  
      个父类函数被调用多次。

一些更深入的问题：各位可以看到，print F.__mro__时发现里面元素的顺序是 F E B C D A
object，这就是F的基类查找顺序，至于为什么是这样的顺序，以及python内置的多继承顺序是怎么实现的，这涉及到mro顺序的实现，python
2.3以后的版本中是采用的一个叫做C3的算法，在下篇博客中进行介绍。

