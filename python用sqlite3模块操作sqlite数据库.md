# python用sqlite3模块操作sqlite数据库

SQLite是一个包含在C库中的轻量级数据库。它并不需要独立的维护进程，并且允许使用非标准变体(nonstandard
variant)的SQL查询语句来访问数据库。

一些应用可是使用SQLite保存内部数据。它也可以在构建应用原型的时候使用，以便于以后转移到更大型的数据库。

SQLite的主要优点：

1\. 一致性的文件格式：

在SQLite的官方文档中是这样解释的，我们不要将SQLite与Oracle或PostgreSQL去比较，与我们自定义格式的数据文件相比，SQLite不仅提
供了很好的

移植性，如大端小端、32/64位等平台相关问题，而且还提供了数据访问的高效性，如基于某些信息建立索引，从而提高访问或排序该类数据的性能，SQLite提供的事
务功能，也是在操作普通文件时无法有效保证的。



2\. 在嵌入式或移动设备上的应用：

由于SQLite在运行时占用的资源较少，而且无需任何管理开销，因此对于PDA、智能手机等

移动设备来说，SQLite的优势毋庸置疑。



3\. 内部数据库：

在有些应用场景中，我们需要为插入到数据库服务器中的数据进行数据过滤或数据清理，以保证最终插入到数据库服务器中的数据有效性。有的时候，数据是否有效，不能通过单
一一条记录来进行判断，而是需要和之前一小段时间的历史数据进行特殊的计算，再通过计算的结果判断当前的数据是否合法。

在这种应用中，我们可以用SQLite缓冲这部分历史数据。还有一种简单的场景也适用于SQLite，即统计数据的预计算。比如我们正在运行数据实时采集的服务程序，
我们可能需要将每10秒的数据汇总后，形成每小时的统计数据，该统计数据可以极大的减少用户查询时的数据量，从而大幅提高前端程序的查询效率。在这种应用中，我们可以
将1小时内的采集数据均缓存在SQLite中，在达到整点时，计算缓存数据后清空该数据。



4\. 数据分析：

可以充分利用SQLite提供SQL特征，完成简单的数据统计分析的功能。这一点是yaml,csv文件无法比拟的。



用我的话来说，他很小，很适合做临时的数据库，迁移数据很简单，直接传递文件就可以了。
其实我一开是是选用leveldb的，但是他的特性像nosql，一些稍微复杂的查询，就有些麻烦了。

1、创建一个新的数据库：sqlite3     文件名

这个test.db 存放着所有的数据。

sqlite3  rui.db



2、打开一个已经存在的数据库：sqlite3      已经存在的文件名

创建一个新数据库和打开一个已经存在的数据库命令是一模一样的，如果文件在当前目录下不存在，则新建；如果存在，则打开。



3、导入数据：.read     数据文件

打开记事本，并将下列 SQL 语句复制到记事本中，保存为 test.sql 到上面说到的 Db 目录下，在命令行环境中输入

.read   test.sql

即将所有的数据导入到 rui.db 数据库中。

 4、列出所有的数据表： .tables

完成上面所有的工作以后，我们就可以列出所有的数据表了

    
    
    [root@devops-ruifengyun /tmp ]$ sqlite3 rui.db 
    SQLite version 3.7.17 2013-05-20 00:56:22
    Enter ".help" for instructions
    Enter SQL statements terminated with a ";"
    sqlite> .tables
    ceshi  tbl1 
    sqlite> 
    sqlite>

5、显示数据库结构：.schema

其实就是一些 SQL 语句，他们描述了数据库的结构，如图

    
    
    sqlite> .schema
    CREATE TABLE tbl1(one varchar(10), two smallint);
    CREATE TABLE ceshi (user text, note text);

6、显示表的结构：.schema    表名

    
    
    sqlite> .schema ceshi
    CREATE TABLE ceshi (user text, note text)

7、导出某个表的数据： .dump    表名

    
    
    sqlite> .dump tbl1
    PRAGMA foreign_keys=OFF;
    BEGIN TRANSACTION;
    CREATE TABLE tbl1(one varchar(10), two smallint);
    INSERT INTO "tbl1" VALUES('goodbye',20);
    INSERT INTO "tbl1" VALUES('hello!',10);
    COMMIT;

再来讲解下python sqlite3的用法，其实和mysqldb很像吧，他的语法和mysql差不多

    
    
    import sqlite3
    #原文： xiaorui.cc 
    #链接数据库文，sqlite都是以文件的形式存在的。
    #如果数据库文件不存在，回新建一个，如果存在则打开此文件
    conn = sqlite3.connect('example')
    c = conn.cursor()
    #创建table
    c.execute('''create table ceshi (user text, note text)''')
      
    # 插入数据，执行SQL语句
    c.execute('''insert into ceshi (user,note)  values('mPfiJRIH9T','mPfiJRIH9T')''')
    c.execute('''insert into ceshi (user,note)  values('7IYcUrKWbw','7IYcUrKWbw')''')
    c.execute('''insert into ceshi (user,note)  values('bXB9VcPdnq','bXB9VcPdnq')''')
    c.execute('''insert into ceshi (user,note)  values('2JFk7EWcCz','2JFk7EWcCz')''')
    c.execute('''insert into ceshi (user,note)  values('QeBFAlYdPr','QeBFAlYdPr')''')
    c.execute('''insert into ceshi (user,note)  values('bDL4T69rsj','bDL4T69rsj')''')
    c.execute('''insert into ceshi (user,note)  values('BOxPqmkEd9','BOxPqmkEd9')''')
    c.execute('''insert into ceshi (user,note)  values('rvBegjXs16','rvBegjXs16')''')
    c.execute('''insert into ceshi (user,note)  values('CWrhA2eSmQ','CWrhA2eSmQ')''')
    c.execute('''insert into ceshi (user,note)  values('qQicfV2gvG','qQicfV2gvG')''')
    c.execute('''insert into ceshi (user,note)  values('s3vg1EuBQb','s3vg1EuBQb')''')
    c.execute('''insert into ceshi (user,note)  values('Lne4xj3Xpc','Lne4xj3Xpc')''')
    c.execute('''insert into ceshi (user,note)  values('PH3R86CKDT','PH3R86CKDT')''')
    c.execute('''insert into ceshi (user,note)  values('HEK7Ymg0Bw','HEK7Ymg0Bw')''')
    c.execute('''insert into ceshi (user,note)  values('lim2OCxhQp','lim2OCxhQp')''')
    c.execute('''insert into ceshi (user,note)  values('kVFfLljBJI','kVFfLljBJI')''')
    c.execute('''insert into ceshi (user,note)  values('Hpbs3VOXNq','Hpbs3VOXNq')''')
    c.execute('''insert into ceshi (user,note)  values('f5ubmznBIE','f5ubmznBIE')''')
    c.execute('''insert into ceshi (user,note)  values('beJCQA2oXV','beJCQA2oXV')''')
    c.execute('''insert into ceshi (user,note)  values('JyPx0iTBGV','JyPx0iTBGV')''')
    c.execute('''insert into ceshi (user,note)  values('4S7RQTqw2A','4S7RQTqw2A')''')
    c.execute('''insert into ceshi (user,note)  values('ypDgkKi27e','ypDgkKi27e')''')
    c.execute('''insert into ceshi (user,note)  values('Anrwx8SbIk','Anrwx8SbIk')''')
    c.execute('''insert into ceshi (user,note)  values('k5ZJFrd8am','k5ZJFrd8am')''')
    c.execute('''insert into ceshi (user,note)  values('KYcTv54QVC','KYcTv54QVC')''')
    c.execute('''insert into ceshi (user,note)  values('Jv6OyfMA0g','Jv6OyfMA0g')''')
    c.execute('''insert into ceshi (user,note)  values('kpSLsQYzuV','kpSLsQYzuV')''')
    c.execute('''insert into ceshi (user,note)  values('u2zkJQWdOY','u2zkJQWdOY')''')
    c.execute('''insert into ceshi (user,note)  values('D0aspFbW8c','D0aspFbW8c')''')
    c.execute('''insert into ceshi (user,note)  values('CwqhvDOrWZ','CwqhvDOrWZ')''')
    c.execute('''insert into ceshi (user,note)  values('Tdy5LA9sWO','Tdy5LA9sWO')''')
    c.execute('''insert into ceshi (user,note)  values('76HnRVbLX0','76HnRVbLX0')''')
    c.execute('''insert into ceshi (user,note)  values('B3aoFibRPV','B3aoFibRPV')''')
    c.execute('''insert into ceshi (user,note)  values('7Q6lNdL5JP','7Q6lNdL5JP')''')
    c.execute('''insert into ceshi (user,note)  values('Hsob6Jyv4A','Hsob6Jyv4A')''')
    #将变动保存到数据库文件，如果没有执行词语句，则前面的insert 语句操作不会被保存
    conn.commit()
    c.execute('''select * from ceshi ''').fetchall()
    #得到所有的记录
    rec = c.execute('''select * from ceshi''')
    print c.fetchall()

  

