# 纯Python的ODBC库 PyPyODBC 1.0 发布

纯Python的ODBC库PyPyODBC 1.0 发布，新版本同时兼容于Python2.x和Python3.3。

PyPyODBC是一个Python ODBC库，它可以被视为著名的PyODBC库的纯Python实现，它们的用法几乎完全一样——就像是PyPy用Python
山寨了Python，PyPyODBC用Python山寨了PyODBC。

而基于纯Python代码的特质给PyPyODBC库带来极大的兼容性、可嵌入性和代码移植性——PyPyODBC可以运行在CPython，IronPython和
PyPy虚拟机下，可以运行在Windows，Linux平台下，可以运行在Python
3.3、2.4、2.5、2.6、2.7等版本下，可以被嵌入在项目中，而无需在运行环境额外编译和安装Python ODBC库模块。

其他亮点：

简单轻便 - PyPyODBC库只有一个Python脚本文件，代码不超过3000行。你可以很容易就把它嵌入到你的项目中。

内建Access MDB支持 - 在Windows平台上，PyPyODBC即可自行创建Access数据库而无需安装微软Office套件。

代码示例

    
    
    import pypyodbc
    pypyodbc.win_create_mdb('D:\\database.mdb') 
    connection_string = 'Driver={Microsoft Access Driver (*.mdb)};DBQ=D:\\database.mdb'
    connection = pypyodbc.connect(connection_string) 
    SQL = 'CREATE TABLE saleout (id COUNTER PRIMARY KEY,product_name VARCHAR(25))'
    connection.cursor().execute(SQL)
    ...

试用PyPyODBC

如果你有一个使用了PyODBC的脚本，如果想试一试PyPyODBC的效果，你要做的就是在这个脚本中注释掉一行代码，换成另一行代码，就像这样：

    
    
    #import pyodbc
    import pypyodbc as pyodbc

  

