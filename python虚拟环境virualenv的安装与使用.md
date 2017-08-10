# python虚拟环境virualenv的安装与使用

virtualenv
是一个创建隔绝的Python环境的工具。virtualenv创建一个包含所有必要的可执行文件的文件夹，用来使用Python工程所需的包。

在安装完python及pip，setuptools等工具后，即可以创建virualenv虚拟环境了，这个类似于虚拟机的工具，可以让同一台电脑中运行多个不同版
本的python程序，互不影响，不用的时候，可以退出或删除，挺不错的一个开发工具。

### 一、安装virtualenv

    
    
    #安装python
    brew install python
    curl https://bootstrap.pypa.io/ez_setup.py -o - | sudo python
    sudo easy_install pip
    # 使用pip安装virtualenv
    pip install virtualenv

### 二、virtualenv的使用

    
    
    #创建一个叫做pythonEnv的新环境
    virtualenv pythonEnv
    #激活再使用
    cd pythonEnv
    source bin/activate
    #退出环境
    deactivate

### 三，使用virtualenvwrapper管理虚拟环境

安装virtualenvwrapper

    
    
    pip install virtualenvwrapper

配置环境变量：

    
    
    vim ~/.bash_profile
    
    
    # Virtualenv/VirtualenvWrapper
    source /usr/local/bin/virtualenvwrapper.sh

保存退出

然后执行以下命令，让系统重新加载配置

    
    
    source ~/.bash_profile

创建环境

    
    
    mkvirtualenv pythonEnv #在 ~/Envs 中创建 pythonEnv文件夹
    mkvirtualenv python3Env -p python3.5 #创建python3.5的环境

切换环境：

    
    
    workon pythonEnv

退出环境：

    
    
    deactivate

删除环境：

    
    
    rmvirtualenv pythonEnv

### 其他

1、其他命令

lsvirtualenv #列举所有的环境。

cdvirtualenv #导航到当前激活的虚拟环境的目录中，比如说这样你就能够浏览它的 site-packages 。

cdsitepackages #和上面的类似，但是是直接进入到 site-packages 目录中。

lssitepackages #显示 site-packages 目录中的内容。

  

2、使用easy_install命令安装pip的时候，出现ImportError: No module named extern错误

原因：mac自带的python2.7.12的extern模块没有安装

解决办法：

#从<https://pypi.python.org/pypi/extern/0.1.0>  下载extern， 然后解压缩安装

tar zxf extern-0.1.0.tar.gz && python setup.py install

  

