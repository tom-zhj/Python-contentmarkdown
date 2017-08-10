# Python重新加载模块方法

为防止两个模块互相导入的问题，Python默认所有的模块都只导入一次，如果需要重新导入模块，

Python2.7可以直接用reload()，Python3可以用下面几种方法：

  

方法一：基本方法

from imp import reload

reload(module)

  

方法二：按照套路，可以这样

import imp

imp.reload(module)

  

方法三：看看imp.py，有发现，所以还可以这样

import importlib

importlib.reload(module)

  

方法四：根据天理，当然也可以这样

from importlib import reload

reload(module)

  

