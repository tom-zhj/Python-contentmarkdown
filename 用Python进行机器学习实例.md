# 用Python进行机器学习实例

### 概要

本文是用Python编程语言来进行机器学习小实验的第一篇。主要内容如下：

  

读入数据并清洗数据

探索理解输入数据的特点

分析如何为学习算法呈现数据

选择正确的模型和学习算法

评估程序表现的准确性

  

### 读入数据 Reading the data

当读入数据时，你将面临处理无效或丢失数据的问题，好的处理方式相比于精确的科学来说，更像是一种艺术。因为这部分处理适当可以适用于更多的机器学习算法并因此提高成
功的概率。

  

用NumPy有效地咀嚼数据，用SciPy智能地吸收数据

Python是一个高度优化的解释性语言，在处理数值繁重的算法方面要比C等语言慢很多，那为什么依然有很多科学家和公司在计算密集的领域将赌注下在Python上呢
？因为Python可以很容易地将数值计算任务分配给C或Fortran这些底层扩展。其中NumPy和SciPy就是其中代表。NumPy提供了很多有效的数据结构
，比如array，而SciPy提供了很多算法来处理这些arrays。无论是矩阵操作、线性代数、最优化问题、聚类，甚至快速傅里叶变换，该工具箱都可以满足需求。

![](http://mmbiz.qpic.cn/mmbiz_jpg/BP6HQN6ROpyDvwlAG8co8vvzxvpnxrtYIofa5yYq5xs
cPYEaKe8rRdGOsjib76nU8egeDnsWeXaTgBSDRQWyuRA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&
wx_lazy=1)

### 读入数据操作

这里我们以网页点击数据为例，第一维属性是小时，第二维数据是点击个数。

    
    
    import scipy as sp
    data = sp.genfromtxt('web_traffic.tsv', delimiter='\t')

### 预处理和清洗数据

当你准备好了你的数据结构用于存储处理数据后，你可能需要更多的数据来确保预测活动，或者拥有了很多数据，你需要去思考如何更好的进行数据采样。在将原始数据（raw
data）进行训练之前，对数据进行提炼可以起到很好的作用，有时，一个用提炼的数据的简单的算法要比使用原始数据的高级算法的表现效果要好。这个工作流程被称作特征
工程（feature engineering）。Creative and intelligent that you are, you will
immediately see the results。

  

由于数据集中可能还有无效数值（nan），我们可以事先看一下无效值的个数：

    
    
    hours = data[:,0]
    hits = data[:,1]
    sp.sum(sp.isnan(hits))

  

用下面的方法将其过滤掉：

    
    
    #cleaning the data
    hours = hours[~sp.isnan(hits)]
    hits = hits[~sp.isnan(hits)]

为了将数据给出一个直观的认识，用Matplotlib的pyplot包来将数据呈现出来。

    
    
    import matplotlib.pyplot as plt
    plt.scatter(hours,hits)
    plt.title("Web traffic over the last month")
    plt.xlabel("Time")
    plt.ylabel("Hits/hour")
    plt.xticks([w*7*24 for w in range(10)],
     ['week %i'%w for w in range(10)])
    plt.autoscale(tight=True)
    plt.grid()
    plt.show()

其显示效果如下：

![](http://mmbiz.qpic.cn/mmbiz_jpg/BP6HQN6ROpyDvwlAG8co8vvzxvpnxrtYnic0CI1FcyN
qOShra3XyBkqG6s3LicDmW8cQ2U53BtibcvyTdGjxdibWHw/640?wx_fmt=jpeg&tp=webp&wxfrom
=5&wx_lazy=1)

### 选择合适的学习算法

选择一个好的学习算法并不是从你的工具箱中的三四个算法中挑选这么简单，实际上有更多的算法你可能没有见过。所以这是一个权衡不同的性能和功能需求的深思熟虑的过程，
比如执行速度和准确率的权衡，,可扩展性和易用性的平衡。

  

现在，我们已经对数据有了一个直观的认识，我们接下来要做的是找到一个真实的模型，并且能推断未来的数据走势。

  

用逼近误差（approximation error）来选择模型

在很多模型中选择一个正确的模型，我们需要用逼近误差来衡量模型预测性能，并用来选择模型。这里，我们用预测值和真实值差值的平方来定义度量误差：

    
    
    def error(f, x, y):
        return sp.sum((f(x)-y)**2)

其中f表示预测函数。

用简单直线来拟合数据

我们现在假设该数据的隐含模型是一条直线，那么我们还如何去拟合这些数据来使得逼近误差最小呢？SciPy的polyfit()函数可以解决这个问题，给出x和y轴的
数据，还有参数order（直线的order是1），该函数给出最小化逼近误差的模型的参数。

    
    
    fp1, residuals, rank, sv, rcond = sp.polyfit(hours, hits, 1, full=True)

fp1是polyfit函数返回模型参数，对于直线来说，它是直线的斜率和截距。

如果polyfit的参数full为True的话，将得到拟合过程中更多有用的信息，这里只有residuals是我们感兴趣的，它正是该拟合直线的逼近误差。

然后将该线在图中画出来：

    
    
    #fit straight line model
    fp1, residuals, rank, sv, rcond = sp.polyfit(hours, hits, 1, full=True)
    fStraight = sp.poly1d(fp1)
    #draw fitting straight line
    fx = sp.linspace(0,hours[-1], 1000) # generate X-values for plotting
    plt.plot(fx, fStraight(fx), linewidth=4)
    plt.legend(["d=%i" % fStraight.order], loc="upper left")

用更高阶的曲线来拟合数据

用直线的拟合是不是很好呢？用直线拟合的误差是317,389,767.34，这说明我们的预测结果是好还是坏呢？我们不妨用更高阶的曲线来拟合数据，看是不是能得到
更好的效果。

  

其逼近误差为：

    
    
    Error of straight line: 317389767.34
    Error of Curve2 line: 179983507.878
    Error of Curve3 line: 139350144.032
    Error of Curve10 line: 121942326.364
    Error of Curve50 line: 109504587.153



这里我们进一步看一下实验结果，看看我们的预测曲线是不是很好的拟合数据了呢？尤其是看一下多项式的阶数从10到50的过程中，模型与数据贴合太紧，这样模型不但是去
拟合数据背后的模型，还去拟合了噪声数据，导致曲线震荡剧烈，这种现象叫做 过拟合 。

  

### 小结

从上面的小实验中，我们可以看出，如果是直线拟合的话就太简单了，但多项式的阶数从10到50的拟合又太过了，那么是不是2、3阶的多项式就是最好的答案呢？但我们同
时发现，如果我们以它们作为预测的话，那它们又会无限制增长下去。所以，我们最后反省一下，看来我们还是没有真正地理解数据。

  

衡量性能指标

作为一个ML的初学者，在衡量学习器性能方面会遇到很多问题或错误。如果是拿你的训练数据来进行测试的话，这可能是一个很简单的问题；而当你遇到的不平衡的训练数据时
，数据就决定了预测的成功与否。

  

### 回看数据

我们再仔细分析一下数据，看一下再week3到week4之间，好像是有一个明显的拐点，所以我们把week3.5之后的数据分离出来，训练一条新的曲线。

    
    
    inflection = 3.5*7*24 #the time of week3.5 is an inflection
    time1 = hours[:inflection]
    value1 = hits[:inflection]
    time2 = hours[inflection:]
    value2 = hits[inflection:]
    fStraight1p = sp.polyfit(time1,value1,1)
    fStraight1 = sp.poly1d(fStraight1p)
    fStraight2p = sp.polyfit(time2,value2,1)
    fStraight2 = sp.poly1d(fStraight2p)

显然，这两条直线更好的描述了数据的特征，虽然其逼近误差还是比那些高阶多项式曲线的误差要大，但是这种方式的拟合可以更好的获取数据的发展趋势。相对于高阶多项式曲
线的过拟合现象，对于低阶的曲线，由于没有很好的描述数据，而导致欠拟合的情形。所以为了更好的描述数据特征，使用2阶曲线来拟合数据，来避免过拟合和欠拟合现象的发
生。

  

### 训练与测试

我们训练得到了一个模型，这里就是我们拟合的两个曲线。为了验证我们训练的模型是否准确，我们可以在最初训练时将一部分训练数据拿出来，当做测试数据来使用，而不仅仅
通过逼近误差来判别模型好坏。

  

### 总结

这一小节作为机器学习小实验的引入，主要传递两点意思：

1、要训练一个学习器，必须理解和提炼数据，将注意力从算法转移到数据上

2、学习如何进行机器学习实验，不要混淆训练和测试数据

  

