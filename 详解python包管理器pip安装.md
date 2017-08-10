# 详解python包管理器pip安装

pip对于使用python的朋友并不陌生，当你想安装python模块的时候一定会首先想到它。pip 是一个安装和管理 Python 包的工具 , 是
easy_install 的一个替换品。

今天来说一下，pip的安装方法。

方法一：脚本安装

    
    
    $ wget https://raw.github.com/pypa/pip/master/contrib/get-pip.py
    $ [sudo] python get-pip.py

方法二：源码安装：

    
    
    $ curl -O https://pypi.python.org/packages/source/p/pip/pip-X.X.tar.gz
    $ tar xvfz pip-X.X.tar.gz
    $ cd pip-X.X
    $ python setup.py install

但是安装过程可能会出现错误：

An error occurred while trying to run get-pip.py. Make sure you have
setuptools or distribute installed.

  

出现这个错误，说明首先要安装setuptools

  

setuptools 安装：

    
    
    wget -q http://peak.telecommunity.com/dist/ez_setup.py
    python ez_setup.py

安装完setuptools后，再次源码安装就好了。

  

