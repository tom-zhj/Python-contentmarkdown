# python数据库连接池技术总结

如果你在开线程请求数据库中如果你觉得所用时间太长的话，你就可以通过python数据库连接池去改善一下在此方面的不足之处，以下是文章的具体介绍，你可以通过我们
的文章对python数据库连接池有一个更好的了解。

昨天测试了一下开500个线程去请求数据库，不过这个时间不清楚会耗多少。即同时发起这么多的线程其效率会如何。于是想到是不是用数据库连接池技术可以明显改善一下这
样的连接操作呢。呆会整理完了之后要测试一个数据：频繁建立与关闭数据库连接的效率与连接池之间的性能对比！

一、DBUtils模块学习

DBUtils实际上是一个包含两个子模块的Python包，一个用于连接DB-API 2模块，另一个用于连接典型的PyGreSQL模块。全局的DB-API
2变量

SteadyDB.py

用于稳定数据库连接

PooledDB.py

连接池

PersistentDB.py

维持持续的数据库连接（持续性连接）

SimplePooledDB.py

简单连接池PS：先摘抄DB-API出来一下吧

<!--[if !vml]--><!--[endif]-->

安装为顶层模块来的两个模块提供基本服务， PersistentDB 和 PooledDB 。

DBUtils.PersistentDB 实现了强硬的、线程安全的、顽固的数据库连接，使用DB-API 2模块。如下图展示了使用 PersistentDB
时的连接层步骤：DBUtils.PooledDB 实现了一个强硬的、线程安全的、有缓存的、可复用的数据库连接，使用任何DB-API 2模块。如下图展示了使用
PooledDB 时的工作流程：

目前供我们选择的有两个模块：PersistentDB 和 PooledDB 都是为了重用数据库连接来提高性能，并保持数据库的稳定性。

python setup.py install

具体的模块学习：

DBUtils.SimplePooledDB 是一个非常简单的数据库连接池实现。他比完善的 PooledDB 模块缺少很多功能。
DBUtils.SimplePooledDB 本质上类似于 MiscUtils.DBPool 这个Webware的组成部分。你可以把它看作一种演示程序

DBUtils.SteadyDB 是一个模块实现了"强硬"的数据库连接，基于DB-API 2建立的原始连接。一个"强硬"的连接意味着在连接关闭之后，或者使用
次数操作限制时会重新连接。一个典型的例子是数据库重启时，而你的程序仍然在运行并需要访问数据库，或者当你的程序连接了一个防火墙后面的远程数据库，而防火墙重启时
丢失了状态时。

一般来说你不需要直接使用 SteadyDB 它只是给接下

  

