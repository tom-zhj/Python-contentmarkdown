# python调用浏览器，实现刷网页小程序

python 打开浏览器，可以做简单的刷网页的小程序 and 其他有想象力的程序。不过仅供学习，勿用非法用途。

python的webbrowser模块支持对浏览器进行一些操作

主要有以下三个方法：

    
    
    webbrowser.open(url, new=0, autoraise=True)
    webbrowser.open_new(url)
    webbrowser.open_new_tab(url)

上面三种方法任意一种都可以，在python2.7下测试通过，不过这个要在windows下测试哦

我们需要了解webbrowser.open()方法：

webbrowser.open(url, new=0, autoraise=True)

在系统的默认浏览器中访问url地址，如果new=0,url会在同一个

浏览器窗口中打开；如果new=1，新的浏览器窗口会被打开;new=2

新的浏览器tab会被打开。

而webbrowser.get()方法可以获取到系统浏览器的操作对象。

webbrowser.register()方法可以注册浏览器类型，而允许被注册的类型名称如下：

    
    
    Type Name Class Name Notes 
    'mozilla' Mozilla('mozilla')   
    'firefox' Mozilla('mozilla')   
    'netscape' Mozilla('netscape')   
    'galeon' Galeon('galeon')   
    'epiphany' Galeon('epiphany')   
    'skipstone' BackgroundBrowser('skipstone')   
    'kfmclient' Konqueror() (1) 
    'konqueror' Konqueror() (1) 
    'kfm' Konqueror() (1) 
    'mosaic' BackgroundBrowser('mosaic')   
    'opera' Opera()   
    'grail' Grail()   
    'links' GenericBrowser('links')   
    'elinks' Elinks('elinks')   
    'lynx' GenericBrowser('lynx')   
    'w3m' GenericBrowser('w3m')   
    'windows-default' WindowsDefault (2) 
    'macosx' MacOSX('default') (3) 
    'safari' MacOSX('safari') (3) 
    'google-chrome' Chrome('google-chrome')   
    'chrome' Chrome('chrome')   
    'chromium' Chromium('chromium')   
    'chromium-browser' Chromium('chromium-browser')

实例：

    
    
    #!/usr/bin/env python
    #-*- coding:UTF-8 -*-
    import webbrowser
    url = 'http://www.pythontab.com'
    webbrowser.open(url)
    print webbrowser.get()

非常简单吧，更深的用途自己去研究哈，点到为止

  

