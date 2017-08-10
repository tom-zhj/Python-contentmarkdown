# python字符串连接的三种方法及其效率、适用场景详解

python字符串连接的方法，一般有以下三种:

### 方法1：直接通过加号(+)操作符连接

    
    
    website = 'python' + 'tab' + '.com'

### 方法2：join方法

    
    
    listStr = ['python', 'tab', '.com'] 
    website = ''.join(listStr)

### 方法3：替换

    
    
    website = '%s%s%s' % ('python', 'tab', '.com')

  

### 下面再来说一下三种方法的不同

方法1，使用简单直接，但是网上不少人说这种方法效率低

之所以说python 中使用 + 进行字符串连接的操作效率低下，是因为python中字符串是不可变的类型，使用 +
连接两个字符串时会生成一个新的字符串，生成新的字符串就需要重新申请内存，当连续相加的字符串很多时(a+b+c+d+e+f+...) ，效率低下就是必然的了

  

方法2，使用略复杂，但对多个字符进行连接时效率高，只会有一次内存的申请。而且如果是对list的字符进行连接的时候，这种方法必须是首选

  

方法3：字符串格式化，这种方法非常常用，本人也推荐使用该方法

  

### 下面用实验来说明字符串连接的效率问题。

    
    
    比较对象：加号连接 VS join连接
    python版本： python2.7
    系统环境：CentOS

**实验一：**
    
    
    # -*- coding: utf-8 -*-
    from time import time
    def method1():
        t = time()
        for i in xrange(100000):
            s = 'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'+'pythontab'
        print time() - t
    def method2():
        t = time()
        for i in xrange(100000):
            s = ''.join(['pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab','pythontab'])
        print time() -t
    method1()
    method2()

结果：

    
    
    0.641695976257
    0.341440916061

  

**实验二：**
    
    
    # -*- coding: utf-8 -*-
    from time import time
    def method1():
        t = time()
        for i in xrange(100000):
            s = 'pythontab'+'pythontab'+'pythontab'+'pythontab'
        print time() - t
    def method2():
        t = time()
        for i in xrange(100000):
            s = ''.join(['pythontab','pythontab','pythontab','pythontab'])
        print time() -t
    method1()
    method2()

结果：

    
    
    0.0265691280365
    0.0522091388702

  

上面两个实验出现了完全不同的结果，分析这两个实验唯一不同的是：字符串连接个数。

**结论：加号连接效率低是在连续进行多个字符串连接的时候出现的，如果连接的个数较少，加号连接效率反而比join连接效率高**

  

