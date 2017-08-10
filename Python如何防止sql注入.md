# Python如何防止sql注入

## 前言

web漏洞之首莫过于sql了，不管使用哪种语言进行web后端开发，只要使用了关系型数据库，可能都会遇到sql注入攻击问题。那么在Python
web开发的过程中sql注入是怎么出现的呢，又是怎么去解决这个问题的？

  

当然，我这里并不想讨论其他语言是如何避免sql注入的，网上关于PHP防注入的各种方法都有，Python的方法其实类似，这里我就举例来说说。

## 起因

漏洞产生的原因最常见的就是字符串拼接了，当然，sql注入并不只是拼接一种情况，还有像宽字节注入，特殊字符转义等等很多种，这里就说说最常见的字符串拼接，这也是
初级程序员最容易犯的错误。

  

首先咱们定义一个类来处理mysql的操作

    
    
    
    class Database:
        hostname = '127.0.0.1'
        user = 'root'
        password = 'root'
        db = 'pythontab'
        charset = 'utf8'
        def __init__(self):
            self.connection = MySQLdb.connect(self.hostname, self.user, self.password, self.db, charset=self.charset)
            self.cursor = self.connection.cursor()
        def insert(self, query):
            try:
                self.cursor.execute(query)
                self.connection.commit()
            except Exception, e:
                print e
                self.connection.rollback()
        def query(self, query):
            cursor = self.connection.cursor(MySQLdb.cursors.DictCursor)
            cursor.execute(query)
            return cursor.fetchall()
        def __del__(self):
            self.connection.close()

这个类有问题吗？

答案是：有！

  

这个类是有缺陷的，很容易造成sql注入，下面就说说为何会产生sql注入。

  

为了验证问题的真实性，这里就写一个方法来调用上面的那个类里面的方法，如果出现错误会直接抛出异常。

    
    
    def test_query(testUrl):
        mysql = Database()
        try:
            querySql = "SELECT * FROM `article` WHERE url='" + testUrl + "'"
            chanels = mysql.query(querySql)
            return chanels
        except Exception, e:
            print e

这个方法非常简单，一个最常见的select查询语句，也使用了最简单的字符串拼接组成sql语句，很明显传入的参数 testUrl
可控，要想进行注入测试，只需要在testUrl的值后面加上单引号即可进行sql注入测试，这个不多说，肯定是存在注入漏洞的，脚本跑一遍，看啥结果

  

(1064, "You have an error in your SQL syntax; check the manual that
corresponds to your MariaDB server version for the right syntax to use near
''t.tips''' at line 1")

回显报错，很眼熟的错误，这里我传入的测试参数是

t.tips'

下面再说一种导致注入的情况，对上面的方法进行稍微修改后

    
    
    def test_query(testUrl):
        mysql = Database()
        try:
            querySql = ("SELECT * FROM `article` WHERE url='%s'" % testUrl)
            chanels = mysql.query(querySql)
            return chanels
        except Exception, e:
            print e

这个方法里面没有直接使用字符串拼接，而是使用了 %s
来代替要传入的参数，看起来是不是非常像预编译的sql？那这种写法能不能防止sql注入呢？测试一下便知道，回显如下

(1064, "You have an error in your SQL syntax; check the manual that
corresponds to your MariaDB server version for the right syntax to use near
''t.tips''' at line 1")

和上面的测试结果一样，所以这种方法也是不行的，而且这种方法并不是预编译sql语句，那么怎么做才能防止sql注入呢？

## 解决

两种方案

  

1> 对传入的参数进行编码转义

  

2> 使用Python的MySQLdb模块自带的方法

  

第一种方案其实在很多PHP的防注入方法里面都有，对特殊字符进行转义或者过滤。

  

第二种方案就是使用内部方法，类似于PHP里面的PDO，这里对上面的数据库类进行简单的修改即可。

  

修改后的代码

    
    
    class Database:
        hostname = '127.0.0.1'
        user = 'root'
        password = 'root'
        db = 'pythontab'
        charset = 'utf8'
        def __init__(self):
            self.connection = MySQLdb.connect(self.hostname, self.user, self.password, self.db, charset=self.charset)
            self.cursor = self.connection.cursor()
        def insert(self, query, params):
            try:
                self.cursor.execute(query, params)
                self.connection.commit()
            except Exception, e:
                print e
                self.connection.rollback()
        def query(self, query, params):
            cursor = self.connection.cursor(MySQLdb.cursors.DictCursor)
            cursor.execute(query, params)
            return cursor.fetchall()
        def __del__(self):
            self.connection.close()

这里 execute
执行的时候传入两个参数，第一个是参数化的sql语句，第二个是对应的实际的参数值，函数内部会对传入的参数值进行相应的处理防止sql注入，实际使用的方法如下

  

preUpdateSql = "UPDATE `article` SET title=%s,date=%s,mainbody=%s WHERE id=%s"

mysql.insert(preUpdateSql, [title, date, content, aid])

这样就可以防止sql注入，传入一个列表之后，MySQLdb模块内部会将列表序列化成一个元组，然后进行escape操作。

  

