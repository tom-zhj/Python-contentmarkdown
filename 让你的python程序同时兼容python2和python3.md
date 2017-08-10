# 让你的python程序同时兼容python2和python3

你只需要对自己的代码稍微做些修改就可以很好的同时支持python2和python3的。下面我将简要的介绍一下如何让自己的python代码如何同时支持pyth
on2和python3。

  

**放弃python 2.6之前的python版本**

python 2.6之前的python版本缺少一些新特性，会给你的迁移工作带来不少麻烦。如果不是迫不得已还是放弃对之前版本的支持吧。

使用 2to3 工具对代码检查

2to3是python自带的一个代码转换工具，可以将python2的代码自动转换为python3的代码。当然，不幸的是转换出的代码并没有对python2的兼
容做任何的处理。所以我们并不真正使用2to3转换出的代码。执行2to3 t.py 查看输出信息，并修正相关问题。

使用python -3执行python程序

2to3 可以检查出很多python2＆3的兼容性问题，但也有很多问题是2to3发现不了的。在加上 -3 参数后，程序在运行时会在控制台上将python2和
python3不一致，同时2to3无法处理的问题提示出来。比如python3和python2中对除法的处理规则做过改变。使用-3参数执行4/2将提示
DeprecationWarning: classic int division 。

from __future__ import

“from __future__ import”后即可使使用python的未来特性了。python的完整future特性可见 __future__
。python3中所有字符都变成了unicode。在python2中unicode字符在定义时需要在字符前面加
u，但在3中则不需要家u，而且在加u后程序会无法编译通过。为了解决该问题可以 “from future import unicode_literals”
，这样python2中字符的行为将和python3中保持一致，python2中定义普通字符将自动识别为unicode。

import问题

python3中“少”了很多python2的包，在大多情况下这些包之是改了个名字而已。我们可以在import的时候对这些问题进行处理。

try:#python2

   from UserDict import UserDict

   #建议按照python3的名字进行import

   from UserDict import DictMixin as MutableMapping

except ImportError:#python3

   from collections import UserDict

   from collections import MutableMapping

使用python3的方式写程序

python2中print是关键字，到了python3中print变成了函数。事实上在python2.6中已经带了print函数，所以对print你直接按照
2to3中给出的提示改为新写法即可。在python3中对异常的处理做了些变化，这个和print类似，直接按照2to3中的提示修改即可。

检查当前运行的python版本

有时候你或许必须为python2和python3写不同的代码，你可以用下面的代码检查当前系统的python版本。

import sys

if sys.version > '3':

   PY3 = True

else:

   PY3 = False

six

six 提供了一些简单的工具用来封装 Python 2 和 Python 3 之间的差异性。我并不太推荐使用six。如果不需要支持python2.6之前的p
ython版本，即使不用six也是比较容易处理兼容性问题的。使用six会让你的代码更像python2而不是python3。

python3的普及需要每位pythoner的推动，或许你还无法立即升级到python3，但请现在就开始写兼容python3的代码，并在条件成熟时升级到py
thon3。

  

