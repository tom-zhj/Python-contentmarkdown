# Python科学计算库Numpy数组的初始化和基本操作

NumPy系统是Python的一种开源的数值计算扩展。这种工具可用来存储和处理大型矩阵，比Python自身的嵌套列表（nested list structu
re)结构要高效的多（该结构也可以用来表示矩阵（matrix））。它包括：1、一个强大的N维数组对象Array；2、比较成熟的（广播）函数库；3、用于整合C
/C++和Fortran代码的工具包；4、实用的线性代数、傅里叶变换和随机数生成函数。numpy和稀疏矩阵运算包scipy配合使用更加方便。

今天我们来看下Numpy数组。

  

### 一. Numpy数组对象

  

Numpy中的多维数组称为ndarray，它有两个组成部分。

数据本身。

描述数据的元数据。

  

它有以下几个属性：

  

ndarray.ndim：数组的维数

ndarray.shape：数组每一维的大小

ndarray.size：数组中全部元素的数量

ndarray.dtype：数组中元素的类型（numpy.int32, numpy.int16, and numpy.float64等）

ndarray.itemsize：每个元素占几个字节

  

在数组的处理过程中，原始数据不受影响，变化的只是元数据。

  

Numpy数组通常是由相同种类的元素组成，即数组中数据类型必须一致。好处是：数组元素类型相同，可轻松确定存储数组所需的空间大小。同时，numpy可运用向量化
运算来处理整个数组。Numpy数组的索引从0开始。

  

例子：

    
    
    >>> import numpy as np
    >>> a = np.arange(15).reshape(3, 5)
    >>> a
    array([[ 0,  1,  2,  3,  4],
           [ 5,  6,  7,  8,  9],
           [10, 11, 12, 13, 14]])
    >>> a.shape
    (3, 5)
    >>> a.ndim
    2
    >>> a.dtype.name
    'int64'
    >>> a.itemsize
    8
    >>> a.size
    15
    >>> type(a)
    <type 'numpy.ndarray'>
    >>> b = np.array([6, 7, 8])
    >>> b
    array([6, 7, 8])
    >>> type(b)
    <type 'numpy.ndarray'>

  

### 二.创建数组：

  

使用array函数讲tuple和list转为array：

    
    
    >>> import numpy as np
    >>> a = np.array([2,3,4])
    >>> a
    array([2, 3, 4])
    >>> a.dtype
    dtype('int64')
    >>> b = np.array([1.2, 3.5, 5.1])
    >>> b.dtype
    dtype('float64')

  

多维数组：

    
    
    >>> b = np.array([(1.5,2,3), (4,5,6)])
    >>> b
    array([[ 1.5,  2. ,  3. ],
           [ 4. ,  5. ,  6. ]])

生成数组的同时指定类型：

    
    
    >>> c = np.array( [ [1,2], [3,4] ], dtype=complex )
    >>> c
    array([[ 1.+0.j,  2.+0.j],
           [ 3.+0.j,  4.+0.j]])

  

生成数组并赋为特殊值：

ones：全1

zeros：全0

empty：随机数，取决于内存情况

    
    
    >>> np.zeros( (3,4) )
    array([[ 0.,  0.,  0.,  0.],
           [ 0.,  0.,  0.,  0.],
           [ 0.,  0.,  0.,  0.]])
    >>> np.ones( (2,3,4), dtype=np.int16 )                # dtype can also be specified
    array([[[ 1, 1, 1, 1],
            [ 1, 1, 1, 1],
            [ 1, 1, 1, 1]],
           [[ 1, 1, 1, 1],
            [ 1, 1, 1, 1],
            [ 1, 1, 1, 1]]], dtype=int16)
    >>> np.empty( (2,3) )                                 # uninitialized, output may vary
    array([[  3.73603959e-262,   6.02658058e-154,   6.55490914e-260],
           [  5.30498948e-313,   3.14673309e-307,   1.00000000e+000]])

生成均匀分布的array：

arange（最小值，最大值，步长）（左闭右开）

linspace（最小值，最大值，元素数量）

    
    
    >>> np.arange( 10, 30, 5 )
    array([10, 15, 20, 25])
    >>> np.arange( 0, 2, 0.3 )                 # it accepts float arguments
    array([ 0. ,  0.3,  0.6,  0.9,  1.2,  1.5,  1.8])
    >>> np.linspace( 0, 2, 9 )                 # 9 numbers from 0 to 2
    array([ 0.  ,  0.25,  0.5 ,  0.75,  1.  ,  1.25,  1.5 ,  1.75,  2.  ])
    >>> x = np.linspace( 0, 2*pi, 100 )        # useful to evaluate function at lots of points

  

### 三.基本运算:

  

整个array按顺序参与运算：

    
    
    >>> a = np.array( [20,30,40,50] )
    >>> b = np.arange( 4 )
    >>> b
    array([0, 1, 2, 3])
    >>> c = a-b
    >>> c
    array([20, 29, 38, 47])
    >>> b**2
    array([0, 1, 4, 9])
    >>> 10*np.sin(a)
    array([ 9.12945251, -9.88031624,  7.4511316 , -2.62374854])
    >>> a<35
    array([ True, True, False, False], dtype=bool)

  

两个二维使用*符号仍然是按位置一对一相乘，如果想表示矩阵乘法，使用dot：

    
    
    >>> A = np.array( [[1,1],
    ...             [0,1]] )
    >>> B = np.array( [[2,0],
    ...             [3,4]] )
    >>> A*B                         # elementwise product
    array([[2, 0],
           [0, 4]])
    >>> A.dot(B)                    # matrix product
    array([[5, 4],
           [3, 4]])
    >>> np.dot(A, B)                # another matrix product
    array([[5, 4],
           [3, 4]])

  

内置函数（min,max,sum)，同时可以使用axis指定对哪一维进行操作：

    
    
    >>> b = np.arange(12).reshape(3,4)
    >>> b
    array([[ 0,  1,  2,  3],
           [ 4,  5,  6,  7],
           [ 8,  9, 10, 11]])
    >>>
    >>> b.sum(axis=0)                            # sum of each column
    array([12, 15, 18, 21])
    >>>
    >>> b.min(axis=1)                            # min of each row
    array([0, 4, 8])
    >>>
    >>> b.cumsum(axis=1)                         # cumulative sum along each row
    array([[ 0,  1,  3,  6],
           [ 4,  9, 15, 22],
           [ 8, 17, 27, 38]])

  

Numpy同时提供很多全局函数

    
    
    >>> B = np.arange(3)
    >>> B
    array([0, 1, 2])
    >>> np.exp(B)
    array([ 1.        ,  2.71828183,  7.3890561 ])
    >>> np.sqrt(B)
    array([ 0.        ,  1.        ,  1.41421356])
    >>> C = np.array([2., -1., 4.])
    >>> np.add(B, C)
    array([ 2.,  0.,  6.])

  

### 四.寻址，索引和遍历：

  

一维数组的遍历语法和Python list类似：

    
    
    >>> a = np.arange(10)**3
    >>> a
    array([  0,   1,   8,  27,  64, 125, 216, 343, 512, 729])
    >>> a[2]
    8
    >>> a[2:5]
    array([ 8, 27, 64])
    >>> a[:6:2] = -1000    # equivalent to a[0:6:2] = -1000; from start to position 6, exclusive, set every 2nd element to -1000
    >>> a
    array([-1000,     1, -1000,    27, -1000,   125,   216,   343,   512,   729])
    >>> a[ : :-1]                                 # reversed a
    array([  729,   512,   343,   216,   125, -1000,    27, -1000,     1, -1000])
    >>> for i in a:
    ...     print(i**(1/3.))
    ...
    nan
    1.0
    nan
    3.0
    nan
    5.0
    6.0
    7.0
    8.0
    9.0

  

多维数组的访问通过给每一维指定一个索引，顺序是先高维再低维：

    
    
    >>> def f(x,y):
    ...     return 10*x+y
    ...
    >>> b = np.fromfunction(f,(5,4),dtype=int)
    >>> b
    array([[ 0,  1,  2,  3],
           [10, 11, 12, 13],
           [20, 21, 22, 23],
           [30, 31, 32, 33],
           [40, 41, 42, 43]])
    >>> b[2,3]
    23
    >>> b[0:5, 1]                       # each row in the second column of b
    array([ 1, 11, 21, 31, 41])
    >>> b[ : ,1]                        # equivalent to the previous example
    array([ 1, 11, 21, 31, 41])
    >>> b[1:3, : ]                      # each column in the second and third row of b
    array([[10, 11, 12, 13],
           [20, 21, 22, 23]])
    When fewer indices are provided than the number of axes, the missing indices are considered complete slices:
    >>>
    >>> b[-1]                                  # the last row. Equivalent to b[-1,:]
    array([40, 41, 42, 43])

…符号表示将所有未指定索引的维度均赋为 ： ，：在python中表示该维所有元素：

    
    
    >>> c = np.array( [[[  0,  1,  2],               # a 3D array (two stacked 2D arrays)
    ...                 [ 10, 12, 13]],
    ...                [[100,101,102],
    ...                 [110,112,113]]])
    >>> c.shape
    (2, 2, 3)
    >>> c[1,...]                                   # same as c[1,:,:] or c[1]
    array([[100, 101, 102],
           [110, 112, 113]])
    >>> c[...,2]                                   # same as c[:,:,2]
    array([[  2,  13],
           [102, 113]])

  

遍历：

如果只想遍历整个array可以直接使用：

    
    
    >>> for row in b:
    ...     print(row)
    ...
    [0 1 2 3]
    [10 11 12 13]
    [20 21 22 23]
    [30 31 32 33]
    [40 41 42 43]

  

但是如果要对每个元素进行操作，就要使用flat属性，这是一个遍历整个数组的迭代器

    
    
    >>> for element in b.flat:
    ...     print(element)
    ...
    0
    1
    2
    3
    10
    11
    12
    13
    20
    21
    22
    23
    30
    31
    32
    33
    40
    41
    42
    43

  

