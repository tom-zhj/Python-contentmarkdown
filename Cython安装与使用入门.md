# Cython安装与使用入门

### 一、Cython是什么?

  

它是一个用来快速生成Python扩展模块(extention module)的工具

它的语法是python语言语法和c语言语法的混血

他比swig更容易编写python的扩展模块

也许你会说swig可以直接通过c的头文件生成扩展模块，但是swig对回调函数的支持不是很好，

另外，如果用swig，很多情况下，你要写额外的代码将输入的参数转换成python对象以及将输出转成python对象，例如如果封装的一个C函数的参数是输入输出
的话，又如如果C函数的参数中有回调函数的话

而在Cython，C里的类型，如int,float,long，char*等都会在必要的时候自动转成python对象，或者从python对象转成C类型，在转换
失败时会抛出异常，这正是Cython最神奇的地方

另外，Cython对回调函数的支持也很好。

总之，如果你有写python扩展模块的需求，那么Cython真的是一个很好的工具

  

### 二、安转cython

  

cython 在linux下安装：

  

#### 1\. 源码包安装：

    
    
    [blueelwang@pythontab ~]$ wget https://pypi.python.org/packages/b7/67/7e2a817f9e9c773ee3995c1e15204f5d01c8da71882016cac10342ef031b/Cython-0.25.2.tar.gz
    [blueelwang@pythontab ~]$ tar xzvf Cython-0.25.2.tar.gz
    [blueelwang@pythontab ~]$ cd Cython-0.25.2
    [blueelwang@pythontab ~]$ python setup.py install

  

#### 2\. pip包安装

    
    
    [blueelwang@pythontab ~]$ sudo pip install Cython --install-option="--no-cython-compile"

#### 3\. Ubuntu下安装

    
    
    [blueelwang@pythontab ~]$ sudo apt-get install cython

  

安装后  输入 cython 即可验证是否安装成功

  

### 三、使用

  

#### 1、编写以 .pyx为扩展名的 cython程序，hello.pyx

    
    
    def say_hello_to(name):
        print("Hello %s!" % name)

#### 2、编写python程序 setup.py

其目的是把 hello.pyx程序转化成hello.c ，并编译成so文件

    
    
    from distutils.core import setup
    from distutils.extension import Extension
    from Cython.Distutils import build_ext
    ext_modules = [Extension("hello", ["hello.pyx"])]
    setup(
      name = 'Hello world app',
      cmdclass = {'build_ext': build_ext},
      ext_modules = ext_modules
    )

  

#### 3\. 执行python程序

    
    
    [blueelwang@pythontab ~]$ python setup.py build_ext --inplace

执行的结果会生成两个文件：hello.c 和 hello.so( 用PyObject* 封装好的文件)

  

#### 4\. 用python调用 hello.so,调用文件为test.py

    
    
    import hello
    hello.say_hello_to("hi,cython!!")

  

cython的主要目的是： 简化python调用c语言程序的繁琐封装过程，提高python代码执行速度（C语言的执行速度比python快）

  

