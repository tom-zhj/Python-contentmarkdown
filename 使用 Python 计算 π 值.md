# 使用 Python 计算 π 值

π是一个无数人追随的真正的神奇数字。我不是很清楚一个永远重复的无理数的迷人之处。在我看来，我乐于计算π，也就是计算π的值。因为π是一个无理数，它是无限的。这
就意味着任何对π的计算都仅仅是个近似值。如果你计算100位，我可以计算101位并且更精确。迄今为止，有些人已经选拔出超级计算机来试图计算最精确的π。一些极值
包括 计算π的5亿位。你甚至能从网上找到包含 π的一百亿位的文本文件（注意啦！下载这个文件可能得花一会儿时间，并且没法用你平时使用的记事本应用程序打开。）。
对于我而言，如何用几行简单的Python来计算π才是我的兴趣所在。

![](http://www.pythontab.com/uploadfile/2013/0604/20130604095925291.png)  

你总是可以 使用 math.pi 变量的 。它被 包含在 标准库中， 在你试图自己 计算它之前，你应该去使用它 。 事实上 ， 我们将 用它来计算 精度
。作为 开始， 让我们看 一个 非常直截了当的 计算Pi的 方法 。像往常一样，我将使用Python
2.7，同样的想法和代码可能应用于不同的版本。我们将要使用的大部分算法来自 Pi WikiPedia page并加以实现。让我们看看下面的代码：

    
    
    importsys
    importmath
      
    defmain(argv):
      
        iflen(argv) !=1:
            sys.exit('Usage: calc_pi.py <n>')
      
        print'\nComputing Pi v.01\n'
          
        a=1.0
        b=1.0/math.sqrt(2)
        t=1.0/4.0
        p=1.0
              
        foriinrange(int(sys.argv[1])):
            at=(a+b)/2
            bt=math.sqrt(a*b)
            tt=t-p*(a-at)**2
            pt=2*p
              
            a=at;b=bt;t=tt;p=pt
              
        my_pi=(a+b)**2/(4*t)
        accuracy=100*(math.pi-my_pi)/my_pi
              
        print"Pi is approximately: "+str(my_pi)
        print"Accuracy with math.pi: "+str(accuracy)
          
    if__name__=="__main__":
        main(sys.argv[1:])

  

这是个非常简单的脚本，你可以下载，运行，修改，和随意分享给别人。你能够看到类似下面的输出结果：

![wpid-simple_python_pi_calculation-2013-05-28-12-54.png](http://www.pythontab
.com/uploadfile/2013/0604/20130604095925735.png)

你会发现，尽管 n 大于4 ,我们逼近 Pi 精度却没有多大的提升。 我们可以猜到即使
n的值更大，同样的事情（pi的逼近精度没有提升）依旧会发生。幸运的是，有不止一种方法来揭开这个谜。使用 Python Decimal
（十进制）库，我们可以就可以得到更高精度的值来逼近Pi。让我们来看看库函数是如何使用的。这个简化的版本，可以得到多于11位的数字 通常情况小Python
浮点数给出的精度。下面是Python Decimal 库中的一个例子 ：

![wpid-python_decimal_example-2013-05-28-12-54.png](http://www.pythontab.com/u
ploadfile/2013/0604/20130604095925610.png)

看到这些数字。不对！ 我们输入的仅是 3.14，为什么我们得到了一些垃圾(junk)？ 这是内存垃圾(memory junk)。
在坚果壳，Python给你你想要的十进制数，再加上一点点额外的值。 只要精度小于垃圾号码开始时，它不会影响任何计算只要精度小于前面的垃圾号码(junk
number)开始时。 您可以指定你想要多少位数的通过设置getcontext().prec 。我们试试。

![wpid-python_decimal_example_6_digits-2013-05-28-12-54.png](http://www.python
tab.com/uploadfile/2013/0604/20130604095925998.png)

很好。 现在让我们 试着用这个 来 看看我们是否能 与我们以前的 代码 有更好的 逼近 。 现在， 我通常 是反对 使用“ from library
import * ” ， 但在这种情况下， 它会 使代码 看起来更漂亮 。

    
    
    importsys
    importmath
    fromdecimalimport*
      
    defmain(argv):
      
        iflen(argv) !=1:
            sys.exit('Usage: calc_pi.py <n>')
      
        print'\nComputing Pi v.01\n'
          
        a=Decimal(1.0)
        b=Decimal(1.0/math.sqrt(2))
        t=Decimal(1.0)/Decimal(4.0)
        p=Decimal(1.0)
              
        foriinrange(int(sys.argv[1])):
            at=Decimal((a+b)/2)
            bt=Decimal(math.sqrt(a*b))
            tt=Decimal(t-p*(a-at)**2)
            pt=Decimal(2*p)
              
            a=at;b=bt;t=tt;p=pt
              
        my_pi=(a+b)**2/(4*t)
        accuracy=100*(Decimal(math.pi)-my_pi)/my_pi
              
        print"Pi is approximately: "+str(my_pi)
        print"Accuracy with math.pi: "+str(accuracy)
          
    if__name__=="__main__":
        main(sys.argv[1:])

  

输出结果:

![wpid-python_pi_accuracy-2013-05-28-12-54.png](http://www.pythontab.com/uploa
dfile/2013/0604/20130604095925515.png)

好了。我们更准确了，但看起来似乎有一些舍入。从n = 100和n =
1000，我们有相同的精度。现在怎么办？好吧，现在我们来求助于公式。到目前为止，我们计算Pi的方式是通过对几部分加在一起。我从DAN 的关于
Calculating Pi 的文章中发现一些代码。他建议我们用以下3个公式：

Bailey–Borwein–Plouffe 公式

Bellard的公式

Chudnovsky 算法

让我们从Bailey–Borwein–Plouffe 公式开始。它看起来是这个样子：

![wpid-48f7653d58f4ad747327d271ed789415-2013-05-28-12-54.png](http://www.pytho
ntab.com/uploadfile/2013/0604/20130604095926710.png)

在代码中我们可以这样编写它：

    
    
    import sys
    import math
    from decimal import *
      
    def bbp(n):
        pi=Decimal(0)
        k=0
        while k &lt; n:
            pi+=(Decimal(1)/(16**k))*((Decimal(4)/(8*k+1))-(Decimal(2)/(8*k+4))-(Decimal(1)/(8*k+5))-(Decimal(1)/(8*k+6)))
            k+=1
        return pi
      
    def main(argv):
      
            if len(argv) !=2:
            sys.exit('Usage: BaileyBorweinPlouffe.py <prec> <n>')
              
        getcontext().prec=(int(sys.argv[1]))
        my_pi=bbp(int(sys.argv[2]))
        accuracy=100*(Decimal(math.pi)-my_pi)/my_pi
      
        print"Pi is approximately "+str(my_pi)
        print"Accuracy with math.pi: "+str(accuracy)
          
    if __name__=="__main__":
        main(sys.argv[1:])

  

抛开“ 包装”的代码，BBP（N）的功能是你真正想要的。你给它越大的N和给 getcontext().prec
设置越大的值，你就会使计算越精确。让我们看看一些代码结果：

![wpid-python_BaileyBorweinPlouffe-2013-05-28-12-54.png](http://www.pythontab.
com/uploadfile/2013/0604/20130604095926345.png)

这有许多数字位。你可以看出，我们并没有比以前更准确。所以我们需要前进到下一个公式，贝拉公式，希望能获得更好的精度。它看起来像这样：

![wpid-dbf2d4355c108f6b3388985be4976799-2013-05-28-12-54.png](http://www.pytho
ntab.com/uploadfile/2013/0604/20130604095926718.png)

我们将只改变我们的变换公式，其余的代码将保持不变。[点击这里下载Python实现的贝拉公式](http://thelivingpearl.com/2013/
05/28/computing-pi-with-python/download%20Bellard%E2%80%99s%20formula%20in%20p
ython)。让我们看一看bellards(n)：

    
    
    def bellard(n):
       pi=Decimal(0)
       k=0
       while k &lt; n:
           pi+=(Decimal(-1)**k/(1024**k))*( Decimal(256)/(10*k+1)+Decimal(1)/(10*k+9)-Decimal(64)/(10*k+3)-Decimal(32)/(4*k+1)-Decimal(4)/(10*k+5)-Decimal(4)/(10*k+7)-Decimal(1)/(4*k+3))
           k+=1
       pi=pi*1/(2**6)
       return pi

  

输出结果：

![wpid-bellards_pi_run-2013-05-28-12-54.png](http://www.pythontab.com/uploadfi
le/2013/0604/20130604095927473.png)

哦，不，我们得到的是同样的精度。好吧，让我们试试第三个公式， Chudnovsky 算法，它看起来是这个样子：

![wpid-826dc7788dba249ee86fc0135e06b035-2013-05-28-12-54.png](http://www.pytho
ntab.com/uploadfile/2013/0604/20130604095928544.png)

再一次，让我们看一下这个计算公式（假设我们有一个阶乘公式）。[ 点击这里可下载用 python 实现的 Chudnovsky
公式](http://thelivingpearl.com/2013/05/28/computing-pi-with-
python/download%20Chudnovsky%20formula%20in%20python)。

下面是程序和输出结果：

    
    
    def chudnovsky(n):
        pi=Decimal(0)
        k=0
        while k &lt; n:
            pi+=(Decimal(-1)**k)*(Decimal(factorial(6*k))/((factorial(k)**3)*(factorial(3*k)))*(13591409+545140134*k)/(640320**(3*k)))
            k+=1
        pi=pi*Decimal(10005).sqrt()/4270934400
        pi=pi**(-1)
        return pi

![wpid-chudnovsky_pyhton-2013-05-28-12-54.png](http://www.pythontab.com/upload
file/2013/0604/20130604095928766.png)

所以我们有了什么结论？花哨的算法不会使机器浮点世界达到更高标准。我真的很期待能有一个比我们用求和公式时所能得到的更好的精度。我猜那是过分的要求。如果你真的需
要用PI，就只需使用math.pi变量了。然而，作为乐趣和测试你的计算机真的能有多快，你总是可以尝试第一个计算出Pi的百万位或者更多位是几。

  

