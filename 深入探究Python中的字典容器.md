# 深入探究Python中的字典容器

### 字典（dictionary）

我们都曾经使用过语言词典来查找不认识的单词的定义。语言词典针对给定的单词（比如
python）提供一组标准的信息。这种系统将定义和其他信息与实际的单词关联（映射）起来。使用单词作为键定位器来寻找感兴趣的信息。这种概念延伸到 Python
编程语言中，就成了特殊的容器类型，称为 字典（dictionary）。

字典（dictionary） 数据类型在许多语言中都存在。它有时候称为关联 数组（因为数据与一个键值相关联），或者作为散列表。但是在 Python
中，字典（dictionary） 是一个很好的对象，因此即使是编程新手也很容易在自己的程序中使用它。按照正式的说法，Python 中的
字典（dictionary） 是一种异构的、易变的映射容器数据类型。

### 创建字典

本系列中前面的文章介绍了 Python 编程语言中的一些容器数据类型，包括 tuple、string 和 list（参见
参考资料）。这些容器的相似之处是它们都是基于序列的。这意味着要根据元素在序列中的位置访问这些集合中的元素。所以，给定一个名为 a
的序列，就可以使用数字索引（比如 a[0] ）或片段（比如 a[1:5]）来访问元素。Python 中的 字典（dictionary）
容器类型与这三种容器类型的不同之处在于，它是一个无序的集合。不是按照索引号，而是使用键值来访问集合中的元素。这意味着构造字典（dictionary）容器比
tuple、string 或 list 要复杂一些，因为必须同时提供键和相应的值，如清单 1 所示。

### 清单 1. 在 Python 中创建字典，第 1 部分

    
    
    >>> d = {0: 'zero', 1: 'one', 2 : 'two', 3 : 'three', 4 : 'four', 5: 'five'}
    >>> d
    {0: 'zero', 1: 'one', 2: 'two', 3: 'three', 4: 'four', 5: 'five'}
    >>> len(d)
    >>> type(d)     # Base object is the dict class
    <type 'dict'>
    >>> d = {}      # Create an empty dictionary
    >>> len(d)
    >>> d = {1 : 'one'} # Create a single item dictionary
    >>> d
    {1: 'one'}
    >>> len(d)
    >>> d = {'one' : 1} # The key value can be non-numeric
    >>> d
    {'one': 1}
    >>> d = {'one': [0, 1,2 , 3, 4, 5, 6, 7, 8, 9]}
    >>> d
    {'one': [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]}

  

如这个例子所示，在 Python 中创建字典（dictionary）要使用花括号和以冒号分隔的键-值组合。如果没有提供键-值组合，那么就会创建一个空的
dictionary。使用一个键-值组合，就会创建具有一个元素的 dictionary，以此类推，直至您需要的任何规模。与任何容器类型一样，可以使用内置的
len 方法查明集合中元素的数量。

前面的示例还演示了关于字典（dictionary）容器的另一个重要问题。键并不限制为整数；它可以是任何不易变的数据类型，包括
integer、float、tuple 或 string。因为 list
是易变的，所以它不能作为字典（dictionary）中的键。但是字典（dictionary）中的值可以是任何数据类型的。

最后，这个示例说明了 Python 中字典（dictionary）的底层数据类型是 dict 对象。要进一步了解如何使用 Python 中的
字典（dictionary），可以使用内置的帮助解释器来了解 dict 类，如清单 2 所示。

### 清单 2. 获得关于字典（dictionary）的帮助

    
    
    >>> help(dict)on class dict in module __builtin__:
       dict(object)
    | dict() -> new empty dictionary.
    | dict(mapping) -> new dictionary initialized from a mapping object's
    |   (key, value) pairs.
    | dict(seq) -> new dictionary initialized as if via:
    |   d = {}
    |   for k, v in seq:
    |     d[k] = v
    | dict(**kwargs) -> new dictionary initialized with the name=value pairs
    |   in the keyword argument list. For example: dict(one=1, two=2)
    | 
    | Methods defined here:
    | 
    | __cmp__(...)
    |   x.__cmp__(y) <==> cmp(x,y)
    | 
    | __contains__(...)
    |   x.__contains__(y) <==> y in x
    | 
    | __delitem__(...)
    |   x.__delitem__(y) <==> del x[y]
    ...

  

关于 dict 类的帮助指出，可以使用构造函数直接创建字典（dictionary），而不使用花括号。既然与其他容器数据类型相比，在创建字典（dictiona
ry）时必须提供更多的数据，那么这些创建方法比较复杂也就不足为奇了。但是，在实践中使用字典（dictionary）并不难，如清单 3 所示。

### 清单 3. 在 Python 中创建字典（dictionary），第 2 部分

    
    
    >>> l = [0, 1,2 , 3, 4, 5, 6, 7, 8, 9] 
    >>> d = dict(l)(most recent call last):
     File "<stdin>", line 1, in ?: can't convert dictionary 
     update sequence element #0 to a sequence
      
    >>> l = [(0, 'zero'), (1, 'one'), (2, 'two'), (3, 'three')]
    >>> d = dict(l)
    >>> d
    {0: 'zero', 1: 'one', 2: 'two', 3: 'three'}
    >>> l = [[0, 'zero'], [1, 'one'], [2, 'two'], [3, 'three']]
    >>> d
    {0: 'zero', 1: 'one', 2: 'two', 3: 'three'}
    >>> d = dict(l)
    >>> d
    {0: 'zero', 1: 'one', 2: 'two', 3: 'three'}
    >>> d = dict(zero=0, one=1, two=2, three=3) 
    >>> d
    {'zero': 0, 'three': 3, 'two': 2, 'one': 1}
    >>> d = dict(0=zero, 1=one, 2=two, 3=three): keyword can't be an expression

  

可以看到，创建字典（dictionary）需要键值和数据值。第一次从 list 创建字典（dictionary）的尝试失败了，这是因为没有匹配的键-
数据值对。第二个和第三个示例演示了如何正确地创建 字典（dictionary）：在第一种情况下，使用一个 list，其中的每个元素都是一个
tuple；在第二种情况下，也使用一个 list，但是其中的每个元素是另一个 list。在这两种情况下，内层容器都用于获得键到数据值的映射。

直接创建 dict 容器的另一个方法是直接提供键到数据值的映射。这种技术允许显式地定义键和与其对应的值。这个方法其实用处不大，因为可以使用花括号完成相同的任
务。另外，如前面的例子所示，在采用这种方式时对于键不能使用数字，否则会导致抛出一个异常。

访问和修改字典（dictionary）

创建了 dictionary 之后，需要访问其中包含的数据。访问方式与访问任何 Python 容器数据类型中的数据相似，如清单 4 所示。

### 清单 4. 访问 dictionary 中的元素

    
    
    >>> d = dict(zero=0, one=1, two=2, three=3)
    >>> d
    {'zero': 0, 'three': 3, 'two': 2, 'one': 1}
    >>> d['zero']
    >>> d['three']
    >>> d = {0: 'zero', 1: 'one', 2 : 'two', 3 : 'three', 4 : 'four', 5: 'five'}
    >>> d[0]
    'zero'
    >>> d[4]
    'four'
    >>> d[6](most recent call last):
     File "<stdin>", line 1, in ?: 6
    >>> d[:-1](most recent call last):
     File "<stdin>", line 1, in ?: unhashable type

可以看到，从字典（dictionary）中获取数据值的过程几乎与从任何容器类型中获取数据完全一样。在容器名后面的方括号中放上键值。当然，字典（diction
ary）可以具有非数字的键值，如果您以前没有使用过这种数据类型，那么适应这一点需要些时间。因为在字典（dictionary）中次序是不重要的（diction
ary 中数据的次序是任意的），所以可以对其他容器数据类型使用的片段功能，对于
字典（dictionary）是不可用的。试图使用片段或者试图从不存在的键访问数据就会抛出异常，指出相关的错误。

Python 中的字典（dictionary）容器也是易变的数据类型，这意味着在创建它之后可以修改它。如清单 5
所示，可以添加新的键到数据值的映射，可以修改现有的映射，还可以删除映射。

### 清单 5. 修改字典（dictionary）

    
    
    >>> d = {0: 'zero', 1: 'one', 2: 'two', 3: 'three'}
    >>> d[0]
    'zero'
    >>> d[0] = 'Zero'
    >>> d
    {0: 'Zero', 1: 'one', 2: 'two', 3: 'three'}
    >>> d[4] = 'four'
    >>> d[5] = 'five'
    >>> d
    {0: 'Zero', 1: 'one', 2: 'two', 3: 'three', 4: 'four', 5: 'five'}
    >>> del d[0]
    >>> d
    {1: 'one', 2: 'two', 3: 'three', 4: 'four', 5: 'five'}
    >>> d[0] = 'zero'
    >>> d
    {0: 'zero', 1: 'one', 2: 'two', 3: 'three', 4: 'four', 5: 'five'}

  

清单 5 演示了几个重点。首先，修改数据值是很简单的：将新的值分配给适当的键。其次，添加新的键到数据值的映射也很简单：将相关数据分配给新的键值。Python
自动进行所有处理。不需要调用 append 这样的特殊方法。对于 dictionary
容器，次序是不重要的，所以这应该好理解，因为不是在字典（dictionary）后面附加映射，而是将它添加到容器中。最后，删除映射的办法是使用 del
操作符以及应该从容器中删除的键。

在清单 5 中有一个情况看起来有点儿怪，键值是按照数字次序显示的，而且这个次序与插入映射的次序相同。不要误解 —— 情况不总是这样的。Python
字典（dictionary）中映射的次序是任意的，对于不同的 Python 安装可能会有变化，甚至多次使用同一 Python
解释器运行相同代码也会有变化。如果在一个字典（dictionary）中使用不同类型的键和数据值，那么就很容易看出这一点，如清单 6 所示。

### 清单 6. 异构的容器

    
    
    >>> d = {0: 'zero', 'one': 1}   
    >>> d
    {0: 'zero', 'one': 1}
    >>> d[0]
    'zero'
    >>> type(d[0])
    <type 'str'>
    >>> d['one']
    >>> type(d['one'])
    <type 'int'>
    >>> d['two'] = [0, 1, 2] 
    >>> d
    {0: 'zero', 'two': [0, 1, 2], 'one': 1}
    >>> d[3] = (0, 1, 2, 3)
    >>> d
    {0: 'zero', 3: (0, 1, 2, 3), 'two': [0, 1, 2], 'one': 1}
    >>> d[3] = 'a tuple'
    >>> d
    {0: 'zero', 3: 'a tuple', 'two': [0, 1, 2], 'one': 1}

  

如这个例子所示，可以在一个字典（dictionary）中使用不同数据类型的键和数据值。还可以通过修改字典（dictionary）添加新的类型。最后，产生的
dictionary 的次序并不与插入数据的次序匹配。本质上，字典（dictionary）中元素的次序是由 Python
字典（dictionary）数据类型的实际实现控制的。新的 Python
解释器很容易改变这一次序，所以一定不要依赖于元素在字典（dictionary）中的特定次序。

### 用字典（dictionary）进行编程

作为正式的 Python 数据类型，字典（dictionary）支持其他较简单数据类型所支持的大多数操作。这些操作包括一般的关系操作符，比如 <、> 和
==，如清单 7 所示。

### 清单 7. 一般关系操作符

    
    
    >>> d1 = {0: 'zero'}
    >>> d2 = {'zero':0}
    >>> d1 < d2
    >>> d2 = d1
    >>> d1 < d2
    >>> d1 == d2
    >>> id(d1)
    >>> id(d2)
    >>> d2 = d1.copy()
    >>> d1 == d2
    >>> id(d1)
    >>> id(d2)

前面的示例创建两个字典（dictionary）并使用它们测试 <
关系操作符。尽管很少以这种方式比较两个字典（dictionary）；但是如果需要，可以这样做。

然后，这个示例将赋值给变量 d1 的字典（dictionary）赋值给另一个变量 d2。注意，内置的 id() 方法对于 d1 和 d2
返回相同的标识符值，这说明这不是复制操作。要想复制字典（dictionary） ，可以使用 copy()
方法。从这个示例中的最后几行可以看出，副本与原来的字典（dictionary）完全相同，但是容纳这字典（dictionary）的变量具有不同的标识符。

在 Python 程序中使用字典（dictionary）时，很可能希望检查字典（dictionary）中是否包含特定的键或值。如清单 8
所示，这些检查很容易执行。

### 清单 8. 条件测试和字典（dictionary）

    
    
    >>> d = {0: 'zero', 3: 'a tuple', 'two': [0, 1, 2], 'one': 1}
    >>> d.keys()
    [0, 3, 'two', 'one']
    >>> if 0 in d.keys():
    ...   print 'True'
    ... 
    >>> if 'one' in d:
    ...   print 'True'
    ... 
    >>> if 'four' in d:
    ...   print 'Dictionary contains four'
    ... elif 'two' in d:
    ...   print 'Dictionary contains two'
    ... contains two

  

测试字典（dictionary）中键或数据值的成员关系是很简单的。dictionary 容器数据类型提供几个内置方法，包括 keys() 方法和
values() 方法（这里没有演示）。这些方法返回一个列表，其中分别包含进行调用的字典（dictionary）中的键或数据值。

因此，要判断某个值是否是字典（dictionary）中的键，应该使用 in 操作符检查这个值是否在调用 keys()
方法所返回的键值列表中。可以使用相似的操作检查某个值是否在调用 values() 方法所返回的数据值列表中。但是，可以使用字典（dictionary）名作为
简写表示法。这是有意义的，因为一般希望知道某个数据值（而不是键值）是否在字典（dictionary）中。

在 “Discover Python, Part 6” 中，您看到了使用 for 循环遍历容器中的元素是多么容易。同样的技术也适用于 Python
字典（dictionary），如清单 9 所示。

### 清单 9. 迭代和字典（dictionary）

    
    
    >>> d = {0: 'zero', 3: 'a tuple', 'two': [0, 1, 2], 'one': 1}
    >>> for k in d.iterkeys():
    ...   print d[k]
    ... tuple
    [0, 1, 2]
    >>> for v in d.itervalues():
    ...   print v
    ... tuple
    [0, 1, 2]
    >>> for k, v in d.iteritems():
    ...   print 'd[',k,'] = ',v
    ... [ 0 ] = zero[ 3 ] = a tuple[ two ] = [0, 1, 2][ one ] = 1

这个示例演示了遍历字典（dictionary）的三种方式：使用从 iterkeys()、itervalues() 或 iteritems() 方法返回的
Python 迭代器。（顺便说一下，可以通过在字典（dictionary）上直接调用适当方法，比如
d.iterkeys()，从而检查这些方法是否返回一个迭代器而不是容器数据类型。）iterkeys() 方法允许遍历字典（dictionary）的键，而
itervalues() 方法允许遍历字典（dictionary）包含的数据值。另一方面，iteritems() 方法允许同时遍历键到数据值的映射。

### 字典（dictionary）：另一种强大的 Python 容器

本文讨论了 Python 字典（dictionary）数据类型。字典（dictionary）是一种异构的、易变的容器，依赖键到数据值的映射（而不是特定的数字
次序）来访问容器中的元素。访问、添加和删除字典（dictionary）中的元素都很简单，而且字典（dictionary）很容易用于复合语句，比如 if
语句或 for 循环。可以在字典（dictionary）中存储所有不同类型的数据，可以按照名称或其他复合键值（比如 tuple）访问这些数据，所以
Python 字典（dictionary）使开发人员能够编写简洁而又强大的编程语句。

  

