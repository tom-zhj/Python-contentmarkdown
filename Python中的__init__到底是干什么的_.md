# Python中的__init__到底是干什么的?

看到Python中有个函数名比较奇特,__init__我知道加下划线的函数会自动运行,但是不知道它存在的具体意义..

  

今天看到<<简明 Python 教程>>第11章 面向对象的编程,中这样介绍它:"给C++/Java/C#程序员的注释

Python中所有的类成员（包括数据成员）都是 公共的 ，所有的方法都是 有效的 。

只有一个例外：如果你使用的数据成员名称以 双下划线前缀 比如__privatevar，Python的名称管理体系会有效地把它作为私有变量。

这样就有一个惯例，如果某个变量只想在类或对象中使用，就应该以单下划线前缀。而其他的名称都将作为公共的，可以被其他类/对象使用。记住这只是一个惯例，并不是Py
thon所要求的（与双下划线前缀不同）。

同样，注意__del__方法与 destructor 的概念类似。"

恍然大悟原来__init__在类中被用做构造函数,固定也写法,看似很死板,其实有道理  

    
    
    def __init__(self, name):
        '''Initializes the person's data.'''
        self.name = name
        print '(Initializing %s)' % self.name
        # When this person is created, he/she
        # adds to the population
        Person.population += 1

name变量属于对象（它使用self赋值）因此是对象的变量

self.name的值根据每个对象指定，这表明了它作为对象的变量的本质。

  

