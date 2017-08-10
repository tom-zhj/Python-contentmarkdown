# Python 中的引用和类属性的理解

最近对Python 的对象引用机制稍微研究了一下，留下笔记，以供查阅。

首先有一点是明确的：「Python 中一切皆对象」。

那么，这到底意味着什么呢？

如下代码：

    
    
    #!/usr/bin/env python
    a = [0, 1, 2] # 来个简单的list
    # 最初，list 和其中各个元素的id 是这样的。
    print 'origin'
    print id(a),a
    for x in a:
        print id(x), x
    print '----------------------'
    # 我们把第一个元素改改
    print 'after change a[0]'
    a[0] = 4
    print id(a),a
    for x in a:
        print id(x), x
    print '----------------------'
    # 我们再把第二个元素改改
    print 'after change a[1]'
    a[1] = 5
    print id(a),a
    for x in a:
        print id(x), x
    print '----------------------'
    # 回头看看直接写个0 ，id是多少
    print 'how about const 0?'
    print id(0), 0

运行结果如下：

    
    
    PastgiftMacbookPro:python pastgift$ ./refTest.py 
    Origin
    4299760200 [0, 1, 2]
    4298181328 0
    4298181304 1
    4298181280 2
    ----------------------
    after change a[0]
    4299760200 [4, 1, 2]
    4298181232 4
    4298181304 1
    4298181280 2
    ----------------------
    after change a[1]
    4299760200 [4, 5, 2]
    4298181232 4
    4298181208 5
    4298181280 2
    ----------------------
    how about const 0?
    4298181328 0

从「Origin」部分来看，list 中各个元素的地址之间都正好相差24，依次指向各自的数据——这让我想到了数组。

当修改a[0] 的值之后，发现，a[0] 的地址发生了变化。也就是说，赋值语句实际上只是让a[0] 重新指向另一个对象而已。此外，还注意到，a[0]
的地址和a[2]的地址相差48（2个24）。

当再次修改a[1] 之后，同样地，a[1] 的地址也发生变化，有趣的是，这次a[1] 的地址和a[0] 的地址又相差24，和原先的a[2]
相差72（3个24）。

最后，当直接把数字0的地址打印出来后，发现它的地址和最开始的a[0] 的地址完全一样。

至此，基本可以说明，就算是list 中的元素，其实也是引用。修改list 中的元素，实际上还是在修改引用而已。

对于Python 中类属性，有人提到过「类属性在同一类及其子类之间共享，修改类属性会影响到同一类及其子类的所有对象」。

听着挺吓人，但仔细研究之后，其实倒也不是什么大不了的事情。

如下代码：

    
    
    #!/usr/bin/env python
    class Bird(object):
        name = 'bird'
        talent = ['fly']
    class Chicken(Bird):
        pass
    bird = Bird();
    bird2 = Bird(); # 同类实例
    chicken = Chicken(); # 子类实例
    # 最开始是这样的
    print 'Original attr'
    print id(bird.name),      bird.name
    print id(bird.talent),    bird.talent
    print id(bird2.name),     bird2.name
    print id(bird2.talent),   bird2.talent
    print id(chicken.name),   chicken.name
    print id(chicken.talent), chicken.talent
    print '----------------------------'
    # 换个名字看看
    bird.name = 'bird name changed!'
    print 'after changing name'
    print id(bird.name),      bird.name
    print id(bird.talent),    bird.talent
    print id(bird2.name),     bird2.name
    print id(bird2.talent),   bird2.talent
    print id(chicken.name),   chicken.name
    print id(chicken.talent), chicken.talent
    print '----------------------------'
    # 洗个天赋试试（修改类属性中的元素）
    bird.talent[0] = 'walk'
    print 'after changing talent(a list)'
    print id(bird.name),      bird.name
    print id(bird.talent),    bird.talent
    print id(bird2.name),     bird2.name
    print id(bird2.talent),   bird2.talent
    print id(chicken.name),   chicken.name
    print id(chicken.talent), chicken.talent
    print '----------------------------'
    # 换个新天赋树（整个类属性全换掉）
    bird.talent = ['swim']
    print 'after reassign talent'
    print id(bird.name),      bird.name
    print id(bird.talent),    bird.talent
    print id(bird2.name),     bird2.name
    print id(bird2.talent),   bird2.talent
    print id(chicken.name),   chicken.name
    print id(chicken.talent), chicken.talent
    print '----------------------------'
    # 洗掉新天赋树（对新来的类属性中的元素进行修改）
    bird.talent[0] = 'dance'
    print 'changing element after reassigning talent'
    print id(bird.name),      bird.name
    print id(bird.talent),    bird.talent
    print id(bird2.name),     bird2.name
    print id(bird2.talent),   bird2.talent
    print id(chicken.name),   chicken.name
    print id(chicken.talent), chicken.talent
    print '----------------------------'

运行结果：

    
    
    PastgiftMacbookPro:python pastgift$ ./changeAttributeTest.py 
    Original attr
    4301998000 bird
    4301857352 ['fly']
    4301998000 bird
    4301857352 ['fly']
    4301998000 bird
    4301857352 ['fly']
    ----------------------------
    after changing name
    4301986984 bird name changed!
    4301857352 ['fly']
    4301998000 bird
    4301857352 ['fly']
    4301998000 bird
    4301857352 ['fly']
    ----------------------------
    after changing talent(a list)
    4301986984 bird name changed!
    4301857352 ['walk']
    4301998000 bird
    4301857352 ['walk']
    4301998000 bird
    4301857352 ['walk']
    ----------------------------
    after reassign talent
    4301986984 bird name changed!
    4301859512 ['swim']
    4301998000 bird
    4301857352 ['walk']
    4301998000 bird
    4301857352 ['walk']
    ----------------------------
    changing element after reassigning talent
    4301986984 bird name changed!
    4301859512 ['dance']
    4301998000 bird
    4301857352 ['walk']
    4301998000 bird
    4301857352 ['walk']
    ----------------------------

在「Origin」的时候，同类对象，子类对象的相同类属性的地址都是相同的——这就是所谓的「共享」。

修改name 之后，只有被修改的对象name
属性发生变化。这是因为对name的赋值操作实际上就是换了一个字符串，重新引用。字符串本身并没有发生变化。所以并没有在同类和子类之间产生互相影响。

接下来，修改talent 中的元素。这时，情况有所改变：同类及其子类的talent
属性都一起跟着变了——这很好理解，因为它们都引用的内存地址都一样，引用的是同一个对象。

再接下来，给talent 重新赋值，也就是改成引用另外一个对象。结果是只有本实例的talent
属性变化了。从内存地址可以看出，本实例和其他实例的talent 属性已经不再指向相同的对象了。就是说「至此，本实例已经是圈外人士了」。

那么，最后再次修改talent 中元素后，对其他实例无影响的结果也是很好理解了。因为已经是「圈外人士」了嘛，我再怎么折腾也都是自己的事情了。

所以，「类属性在同类及其子类之间互相影响」必须有一个前提条件：实例建立后，其类属性从来没有被重新赋值过，即类属性依然指向最初所指向的内存地址。

最后提一下对象属性

如下代码：

    
    
    #!/usr/bin/env python
    class Bird(object):
        def __init__(self):
            self.talent = ['fly']
    bird = Bird()
    bird2 = Bird()
    # 刚开始的情形
    print 'Origin'
    print id(bird.talent), bird.talent
    print id(bird2.talent), bird2.talent
    print '--------------------'
    # 修改其中一个对象的属性
    bird.talent[0] = 'walk'
    print 'after changing attribute'
    print id(bird.talent), bird.talent
    print id(bird2.talent), bird2.talent
    print '--------------------'
    # 作死：两个对象的属性指向同一个内存地址，再修改
    bird.talent = bird2.talent
    bird.talent[0] = 'swim'
    print 'assign to another attribute and change it'
    print id(bird.talent), bird.talent
    print id(bird2.talent), bird2.talent
    print '--------------------'
    运行结果：
    PastgiftMacbookPro:python pastgift$ ./changeAttributeTest2.py 
    Origin
    4299867632 ['fly']
    4299760200 ['fly']
    --------------------
    after changing attribute
    4299867632 ['walk']
    4299760200 ['fly']
    --------------------
    assign to another attribute and change it
    4299760200 ['swim']
    4299760200 ['swim']
    --------------------

由于对象属性就算内容完全一样（刚初始化后的属性内容一般都是一样的），也会分配到完全不同的内存地址上去。所以不存在「同类对象之间影响」的情况。

但如果让一个对象的属性和另一个对象的属性指向同一个地址，两者之间（但也仅限两者之间）便又互相牵连起来。

  

