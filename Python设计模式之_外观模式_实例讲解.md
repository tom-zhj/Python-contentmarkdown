# Python设计模式之"外观模式"实例讲解

Python中设计模式之外观模式主张以分多模块进行代码管理而减少耦合,下面用实例来进行说明。

应用特性：

在很多复杂而小功能需要调用需求时，而且这些调用往往还有一定相关性，即一调用就是一系列的。

结构特性：

把原本复杂而繁多的调用，规划统一到一个入口类中，从此只通过这一个入口调用就可以了。

代码结构示例：

    
    
    class ModuleOne(object):
      def Create(self):
        print 'create module one instance'
      def Delete(self):
        print 'delete module one instance'
    class ModuleTwo(object):
      def Create(self):
        print 'create module two instance'
      def Delete(self):
        print 'delete module two instance'
    class Facade(object):
      def __init__(self):
        self.module_one = ModuleOne()
        self.module_two = ModuleTwo()
      def create_module_one(self):
        self.module_one.Create()
      def create_module_two(self):
        self.module_two.Create()
      def create_both(self):
        self.module_one.Create()
        self.module_two.Create()
      def delete_module_one(self):
        self.module_one.Delete()
      def delete_module_two(self):
        self.module_two.Delete()
      def delete_both(self):
        self.module_one.Delete()
        self.module_two.Delete()

  

有点类似代理模式，不同之处是，外观模式不仅代理了一个子系统的各个模块的功能，同时站在子系统的角度，通过组合子系统各模块的功能，对外提供更加高层的接口，从而在
语义上更加满足子系统层面的需求。

随着系统功能的不断扩张，当需要将系统划分成多个子系统或子模块，以减少耦合、降低系统代码复杂度、提高可维护性时，代理模式通常会有用武之地。

再来看一个例子：

    
    
    class small_or_piece1: 
      def __init__(self): 
        pass 
       
      def do_small1(self): 
        print 'do small 1' 
       
    class small_or_piece_2: 
      def __init__(self): 
        pass 
       
      def do_small2(self): 
        print 'do small 2' 
       
    class small_or_piece_3: 
      def __init__(self): 
        pass 
       
      def do_small3(self): 
        print 'do small 3' 
     
    class outside: 
      def __init__(self): 
        self.__small1 = small_or_piece1() 
        self.__small2 = small_or_piece_2() 
        self.__small3 = small_or_piece_3() 
       
      def method1(self): 
        self.__small1.do_small1()  ##如果这里调用的不只2两函数，作用就显示出来了，可以把原本复杂的函数调用关系清楚化，统一化 
        self.__small2.do_small2() 
         
      def method2(self): 
        self.__small2.do_small2() 
        self.__small3.do_small3() 
     
    if __name__ == '__main__': 
      osd = outside() 
      osd.method1() 
      osd.method2()

结果：

    
    
    do small 1 
    do small 2 
    do small 2 
    do small 3

  

