# Python 与 Javascript 比较

最近由于工作的需要开始开发一些Python的东西，由于之前一直在使用Javascript，所以会不自觉的使用一些Javascript的概念，语法什么的，经常
掉到坑里。我觉得对于从Javascript转到Python，有必要总结一下它们之间的差异。

基本概念

Python和Javascript都是脚本语言，所以它们有很多共同的特性，都需要解释器来运行，都是动态类型，都支持自动内存管理,都可以调用eval（）来执行
脚本等等脚本语言所共有的特性。

然而它们也有很大的区别，Javascript这设计之初是一种客户端的脚本语言，主要应用于浏览器，它的语法主要借鉴了C，而Python由于其“优雅”，“明确”
，“简单”的设计而广受欢迎，被应用于教育，科学计算，web开发等不同的场景中。

编程范式

Python和Javascript都支持多种不同的编程范式，在面向对象的编程上面，它们有很大的区别。Javascript的面向对象是基于原型（prototy
pe）的， 对象的继承是由原型（也是对象）创建出来的，由原型对象创建出来的对象继承了原型链上的方法。而Python则是中规中矩的基于类（class）的继承，
并天然的支持多态（polymophine）。

OO in Pyhton

    
    
    class Employee:
       'Common base class for all employees'
       empCount = 0 ##类成员
     
       def __init__(self, name, salary):
          self.name = name
          self.salary = salary
          Employee.empCount += 1
        
       def displayCount(self):
         print "Total Employee %d" % Employee.empCount
     
       def displayEmployee(self):
          print "Name : ", self.name,  ", Salary: ", self.salary
    ## 创建实例
    ea = Employee("a",1000)
    eb = Employee("b",2000)
    OO in Javascript
    var empCount = 0;
    //构造函数
    function Employee(name, salary){
        this.name = name;
        this.salary = salary;   
        this.empCount += 1;
    }
     
    Employee.prototype.displayCount = function(){
        console.log("Total Employee " + empCount );
    }
     
    Employee.prototype.displayEmployee = function(){
        console.log("Name " + this.name + ", Salary " + this.salary );
    }
    //创建实例
    var ea = new Employee("a",1000);
    var eb = new Employee("b",2000);

因为是基于对象的继承，在Javascript中，我们没有办法使用类成员empCount，只好声明了一个全局变量，当然实际开发中我们会用更合适的scope。注
意Javascript创建对象需要使用new关键字，而Python不需要。

除了原生的基于原型的继承，还有很多利用闭包或者原型来模拟类继承的Javascript OO工具，因为不是语言本身的属性，我们就不讨论了。

线程模型

在Javascript的世界中是没有多线程的概念的，并发使用过使用事件驱动的方式来进行的，
所有的JavaScript程序都运行在一个线程中。在HTML5中引入web worker可以并发的处理任务，但没有改变Javascript单线程的限制。

Python通过thread包支持多线程。

不可改变类型 （immutable type）

在Python中，有的数据类型是不可改变的，也就意味着这种类型的数据不能被修改，所有的修改都会返回新的对象。而在Javascript中所有的数据类型都是可以
改变的。Python引入不可改变类型我认为是为了支持线程安全，而因为Javascript是单线程模型，所以没有必要引入不可改变类型。

当然在Javascript可以定义一个对象的属性为只读。

    
    
    var obj = {};Object.defineProperty(obj, "prop", {
        value: "test",
        writable: false});

在ECMAScript5的支持中，也可以调用Object的freeze方法来是对象变得不可修改。

Object.freeze(obj)

数据类型

Javascript的数据类型比较简单，有object、string、boolean、number、null和undefined，总共六种

Python中一切均为对象，像module、function、class等等都是。

Python有五个内置的简单数据类型bool、int、long、float和complex，另外还有容器类型，代码类型，内部类型等等。

布尔

Javascript有true和false。Python有True和False。它们除了大小写没有什么区别。

字符串

Javascript采用UTF16编码。

Python使用ASCII码。需要调用encode、decode来进行编码转换。使用u作为前缀可以指定字符串使用Unicode编码。

数值

Javascript中所有的数值类型都是实现为64位浮点数。支持NaN(Not a number)，正负无穷大（+/-Infiity）。

Python拥有诸多的数值类型，其中的复数类型非常方便，所以在Python在科研和教育领域很受欢迎。这应该也是其中一个原因吧。Python中没有定义NaN，
除零操作会引发异常。

列表

Javascript内置了array类型（array也是object）

Python的列表（List）和Javascript的Array比较接近，而元组（Tuple）可以理解为不可改变的列表。

除了求长度在Python中是使用内置方法len外，基本上Javascript和Python都提供了类似的方法来操作列表。Python中对列表下标的操作非常灵
活也非常方便，这是Javascript所没有的。例如l[5:-1],l[:6]等等。

字典、哈希表、对象

Javascript中大量的使用{}来创建对象，这些对象和字典没有什么区别，可以使用[]或者.来访问对象的成员。可以动态的添加，修改和删除成员。可以认为对象
就是Javascript的字典或者哈希表。对象的key必须是字符串。

Python内置了哈希表（dictS），和Javascript不同的是，dictS可以有各种类型的key值。

空值

Javascript定义了两种空值。 undefined表示变量没有被初始化，null表示变量已经初始化但是值为空。

Python中不存在未初始化的值，如果一个变量值为空，Python使用None来表示。

Javascript中变量的声明和初始化

    
    
    v1;
    v2 = null;
    var v3;
    var v4 = null;
    var v5 = 'something';

在如上的代码中v1是全局变量，未初始化，值为undefined；v2是全局变量，初始化为空值；v3为局部未初始化变量，v4是局部初始化为空值的变量；v5是局
部已初始化为一个字符处的变量。

Python中变量的声明和初始化

v1 = None

v2 = 'someting'

Python中的变量声明和初始化就简单了许多。当在Python中访问一个不存在的变量时，会抛出NameError的异常。当访问对象或者字典的值不存在的时候，
会抛出AttributeError或者KeyError。因此判断一个值是否存在在Javascript和Python中需要不一样的方式。

Javascript中检查某变量的存在性：

    
    
    if (!v ) {
        // do something if v does not exist or is null or is false
    }
     
    if (v === undefined) {
        // do something if v does not initialized
    }

注意使用!v来检查v是否初始化是有歧义的因为有许多种情况!v都会返回true

Python中检查某变量的存在性：

    
    
    try:
        v
    except NameError
        ## do something if v does not exist

在Python中也可以通过检查变量是不是存在于局部locals（）或者全局globals（）来判断是否存在该变量。

类型检查

Javascript可以通过typeof来获得某个变量的类型：

typeof in Javascript 的例子：

    
    
    typeof 3 // "number"
    typeof "abc" // "string"
    typeof {} // "object"
    typeof true // "boolean"
    typeof undefined // "undefined"
    typeof function(){} // "function"
    typeof [] // "object"
    typeof null // "object"

要非常小心的使用typeof，从上面的例子你可以看到，typeof null居然是object。因为javscript的弱类型特性，想要获得更实际的类型，还
需要结合使用instanceof，constructor等概念。具体请参考这篇文章

Python提供内置方法type来获得数据的类型。

    
    
    >>> type([]) is list
    True
    >>> type({}) is dict
    True
    >>> type('') is str
    True
    >>> type(0) is int
    True

同时也可以通过isinstance()来判断类的类型

    
    
    class A:
        pass
    class B(A):
        pass
    isinstance(A(), A)  # returns True
    type(A()) == A      # returns True
    isinstance(B(), A)    # returns True
    type(B()) == A        # returns False

但是注意Python的class style发生过一次变化，不是每个版本的Python运行上述代码的行为都一样，在old
style中，所有的实例的type都是‘instance’，所以用type方法来检查也不是一个好的方法。这一点和Javascript很类似。

自动类型转换

当操作不同类型一起进行运算的时候，Javascript总是尽可能的进行自动的类型转换，这很方便，当然也很容易出错。尤其是在进行数值和字符串操作的时候，一不小
心就会出错。我以前经常会计算SVG中的各种数值属性，诸如x，y坐标之类的，当你一不小心把一个字符串加到数值上的时候，Javascript会自动转换出一个数值
，往往是NaN，这样SVG就完全画不出来啦，因为自动转化是合法的，找到出错的地方也非常困难。

Python在这一点上就非常的谨慎，一般不会在不同的类型之间做自动的转换。

语法

风格

Python使用缩进来决定逻辑行的结束非常具有创造性，这也许是Python最独特的属性了，当然也有人对此颇具微词，尤其是需要修改重构代码的时候，修改缩进往往
会引起不小的麻烦。

Javascript虽然名字里有Java，它的风格也有那么一点像Java，可是它和Java就好比雷峰塔和雷锋一样，真的没有半毛钱的关系。到时语法上和C比较类
似。这里必须要提到的是coffeescript作为构建与Javascript之上的一种语言，采用了类似Python的语法风格，也是用缩进来决定逻辑行。

Python风格

    
    
    def func(list):
        for i in range(0,len(list)):
            print list[i]
    Javascript风格
    function funcs(list) {
        for(var i=0, len = list.length(); i < len; i++) {
            console.log(list[i]);
        }
    }

从以上的两个代码的例子可以看出，Python确实非常简洁。

作用范围和包管理

Javascript的作用域是由方法function来定义的，也就是说同一个方法内部拥有相同的作用域。这个严重区别与C语言使用{}来定义的作用域。Closu
re是Javascript最有用的一个特性。

Python的作用域是由module，function，class来定义的。

Python的import可以很好的管理依赖和作用域，而Javascript没有原生的包管理机制，需要借助AMD来异步的加载依赖的js文件，requirej
s是一个常用的工具。

赋值逻辑操作符

Javascript使用=赋值，拥有判断相等（==）和全等（===）两种相等的判断。其它的逻辑运算符有&& 和||，和C语言类似。

Python中没有全等，或和与使用的时and 和 or，更接近自然语言。Python中没有三元运算符 A ：B ？C，通常的写法是

（A and B) or C

因为这样写有一定的缺陷，也可以写作

 B if A else C

Python对赋值操作的一个重要的改进是不允许赋值操作返回赋值的结果，这样做的好处是避免出现在应该使用相等判断的时候错误的使用了赋值操作。因为这两个操作符实
在太像了，而且从自然语言上来说它们也没有区别。

++运算符

Python不支持++运算符，没错你再也不需要根据++符号在变量的左右位置来思考到底是先加一再赋值呢还是先赋值再加一。

连续赋值

利用元组（tuple），Python可以一次性的给多个变量赋值

(MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY) = range(7)

函数参数

Python的函数参数支持命名参数和可选参数（提供默认值），使用起来很方便，Javascript不支持可选参数和默认值（可以通过对arguments的解析来
支持）

def info(object, spacing=10, collapse=1):

    ... ...

其它

立即调用函数表达式 （IIFE）

Javascript的一个方便的特性是可以立即调用一个刚刚声明的匿名函数。也有人称之为自调用匿名函数。

下面的代码是一个module模式的例子，使用闭包来保存状态实现良好的封装。这样的代码可以用在无需重用的场合。

    
    
    var counter = (function(){
        var i = 0;
        return {
            get: function(){
                return i;
                },
            set: function( val ){
                i = val;
                },
            increment: function() {
                return ++i;
                }
            };
        }());

Python没有相应的支持。

生成器和迭代器（Generators & Iterator）

在我接触到的Python代码中，大量的使用这样的生成器的模式。

Python生成器的例子

    
    
    # a generator that yields items instead of returning a list
    def firstn(n):
        num = 0
        while num < n:
            yield num
            num += 1
       
    sum_of_first_n = sum(firstn(1000000))

Javascript1.7中引入了一些列的新特性，其中就包括生成器和迭代器。然而大部分的浏览器除了Mozilla（Mozilla基本上是在自己玩，下一代的J
avascript标准应该是ECMAScript5）都不支持这些特性

Javascript1.7 迭代器和生成器的例子。

    
    
    function fib() {
      var i = 0, j = 1;
      while (true) {
        yield i;
        var t = i;
        i = j;
        j += t;
      }
    }；
     
    var g = fib();
    for (var i = 0; i < 10; i++) {
      console.log(g.next());
    }

列表（字典、集合）映射表达式 （List、Dict、Set Comprehension）

Python的映射表达式可以非常方便的帮助用户构造列表、字典、集合等内置数据类型。

下面是列表映射表达式使用的例子：

    
    
    >>> [x + 3 for x in range(4)]
    [3, 4, 5, 6]
    >>> {x + 3 for x in range(4)}
    {3, 4, 5, 6}
    >>> {x: x + 3 for x in range(4)}
    {0: 3, 1: 4, 2: 5, 3: 6}

Javascript1.7开始也引入了Array Comprehension

    
    
    var numbers = [1, 2, 3, 4];
    var doubled = [i * 2 for (i of numbers)];

Lamda表达式 （Lamda Expression ）

Lamda表达式是一种匿名函数，基于著名的λ演算。许多语言诸如C#，Java都提供了对lamda的支持。Pyhton就是其中之一。Javascript没有提
供原生的Lamda支持。但是有第三方的Lamda包。

g = lambda x : x*3

装饰器（Decorators）

Decorator是一种设计模式，大部分语言都可以支持这样的模式，Python提供了原生的对该模式的支持，算是一种对程序员的便利把。

Decorator的用法如下。

    
    
    @classmethod
    def foo (arg1, arg2):
        ....

  

