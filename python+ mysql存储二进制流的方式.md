# python+ mysql存储二进制流的方式

很多时候我们为了管理方便会把依稀很小的图片存入数据库，有人可能会想这样会不会对数据库造成很大的压力，其实大家可以不用担心，因为我说过了，是存储一些很小的图片
，几K的，没有问题的！

  

再者，在这里我们是想讲一种方法，python+ mysql存储二进制流的方式

  

这里用的是Mysqldb，python里面最常用的数据库模块

    
    
    import MySQLdb   
      
    class BlobDataTestor:   
        def __init__ (self):   
            self.conn = MySQLdb.connect(host='localhost',user='',passwd='',db='0')   
      
        def __del__ (self):   
            try:   
                self.conn.close()   
            except :   
                pass    
      
      
        def closedb(self):   
            self.conn.close()   
      
        def setup(self):   
            cursor = self.conn.cursor()   
            cursor.execute( """  
                CREATE TABLE IF NOT EXISTS `Dem_Picture` (  
                `ID` int(11) NOT NULL auto_increment,  
                `PicData` mediumblob,  
                PRIMARY KEY (`ID`)  
                ) ENGINE=MyISAM DEFAULT CHARSET=utf8 AUTO_INCREMENT=4 ;  
                """)   
      
      
      
        def teardown(self):   
            cursor = self.conn.cursor()   
            try:   
                cursor.execute( "Drop Table Dem_Picture" )   
            except:   
                pass    
            # self.conn.commit()    
      
        def testRWBlobData(self):    
        # 读取源图片数据                  
            f = open( "C:\\11.jpg" , "rb" )   
            b = f.read()   
            f.close()   
      
        # 将图片数据写入表    
            cursor = self.conn.cursor()   
            cursor.execute( "INSERT INTO Dem_Picture (PicData) VALUES (%s)" , (MySQLdb.Binary(b)))   
        # self.conn.commit()    
      
        # 读取表内图片数据，并写入硬盘文件    
            cursor.execute( "SELECT PicData FROM Dem_Picture ORDER BY ID DESC limit 1" )   
            d = cursor.fetchone()[0]   
            cursor.close()   
      
            f = open( "C:\\22.jpg" , "wb" )   
            f.write(d)   
            f.close()   
      
    # 下面一句的作用是：运行本程序文件时执行什么操作  
    if __name__ == "__main__":   
      
        test = BlobDataTestor()   
      
        try:   
            test.setup()   
            test.testRWBlobData()   
            test.teardown()   
        finally:   
            test.closedb()

到这里python mysql存储二进制图片的方法就将完了

  

