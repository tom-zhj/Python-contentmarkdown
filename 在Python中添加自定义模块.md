# 在Python中添加自定义模块

一般来说，我们会将自己写的Python模块与python自带的模块分开存放以达到便于维护的目的。那么如何在Python中添加自定义的模块呢？

在解答这个问题之前，我们首先要明确两点：

**1.严格区分包（package）和文件夹。包的定义就是包含__init__.py的文件夹。如果没有__init__.py,那么就是普通的文件夹。**

**2.模块导入写法，注意只要包路径，不要文件夹路径。**

  

Python 运行环境在查找库文件时是对 sys.path 列表进行遍历，如果我们想在运行环境中注册新的类库，主要有以下2种方法：

**1\. 在sys.path列表中添加新的路径。**

**2\. 将库文件复制到sys.path列表中的目录里（如site-packages目录）。**

  

我们可以通过运行一下代码来查看sys.path

    
    
    import sys
    print sys.path

运行结果：

    
    
    ['/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/lib-old', '/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/lib-dynload', '/System/Library/Frameworks/Python.framework/Versions/2.7/Extras/lib/python/PyObjC', '/Library/Python/2.7/site-packages']

这两种办法中第一种比较简单，而且对环境的影响最小。

下面我们来看一下第一种方法具体如何操作：

  

在python安装目录的site-package文件夹中新建pythontab.pth，上面site-
package的路径是：/Library/Python/2.7/site-packages，文件的内容是：需要导入的package所在的文件夹路径。

这样，Python 在遍历已知的库文件目录过程中，如果见到一个 .pth 文件，就会将文件中所记录的路径加入到 sys.path 设置中，这样 .pth
文件说指明的package也就可以被Python运行环境顺利找到， 我们就可以像使用内置模块一样引入自定义模块了。

如果缺省的sys.path中没有含有自己的模块或包的路径，我们也可以使用sys.path.apend方法来动态加入包路径。

  

