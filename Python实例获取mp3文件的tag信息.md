# Python实例获取mp3文件的tag信息

下面利用一个python的实例程序，来学习python。这个程序的目的就是分析出所有MP3文件的Tag信息并输出。

    
    
    import os # 导入os模块，提供文件路径，列出文件等方法
    import sys # 导入sys模块，使用sys.modules获取模块中的所有内容，类似反射的功能
    from UserDict import UserDict # 这个表示从UserDict类中导入UserDict，类似于Java中的 import UserDict.UserDict
     
    def stripnulls(data):
        "一个空字符串的处理函数将所有00字节的内容替换为空字符，病将前后的空字符串去掉"
        # Python中的strip用于去除字符串的首尾字符，同理，lstrip用于去除左边的字符，rstrip用于去除右边的字符。
        return data.replace("\00", "").strip()
     
    class FileInfo(UserDict):
        '''文件基类，存储文件的文件名，继承自UserDict（存储key-value的一个类，可以重写__setitem__，__getitem__方法，
        就可以使用[]）'''
        # self是定义时使用，使用时不需要，如果没有参数，则filename默认None，如果有一个参数的话，参数即为filename
        def __init__(self, filename=None):
            UserDict.__init__(self) # 初始化父类
            self["name"] = filename # 设置name为 filaname
     
    class MP3FileInfo(FileInfo):
        "MP3文件的信息类，用于分析MP3文件和存储信息"
        # tagDataMap 用于存储MP3的Tag信息分别所在位置，( key ： 开始位置，结束位置， 处理函数),
        # stripnulls表示最开始定义的函数
        tagDataMap = {"title" : ( 3, 33, stripnulls),
        "artist" : ( 33, 63, stripnulls),
        "album" : ( 63, 93, stripnulls),
        "year" : ( 93, 97, stripnulls),
        "comment" : ( 97, 126, stripnulls),
        "genre" : (127, 128, ord)}
         
        def __parse(self, filename): # 解析MP3文件
            self.clear()
            try:
                fsock = open(filename, "rb", 0) # 打开文件
                try:
                    # 设置文件读取的指针位置， seek第二个参数，2表示从文件结尾作为参考点，
                    # -128表示还有128字节结尾的点，0表示文件开头做参考点，1表示当前位置做参考点
                    fsock.seek(-128, 2)
                    tagdata = fsock.read(128) # 读取128字节的数据
                finally:
                    fsock.close() # 关闭文件，注意在finally中，出错也需要关闭文件句柄
                if tagdata[:3] == "TAG": # 判断是否是有效的含Tag的MP3文件
                    # 循环取出Tag信息位置信息， 如3, 33, stripnulls，并依次赋给start, end, parseFunc
                    for tag, (start, end, parseFunc) in self.tagDataMap.items():
                        # tagdata[start:end]读出start到end的字节，使用parseFunc处理这些内容
                        self[tag] = parseFunc(tagdata[start:end])
            except IOError: # 如果出现IOError，则跳过继续
                pass
         
        # 重写__setitem__方法，上面的self[tag] = parseFunc(tagdata[start:end])就会使用这个方法,
        # key为tag，itme为parseFunc(tagdata[start:end])
        def __setitem__(self, key, item):
            if key == "name" and item: # 如果key是 name，并且 item不为空
                self.__parse(item) # 解析MP3文件
                # problem here,should out of the if
                # FileInfo.__setitem__(self, key, item) 如果使用这个缩进就会出现错误
            # 之前的错误点，注意这儿的缩进，无论如何都会存储key-value，使用FileInfo.__setitem__父类的方法来存储
            FileInfo.__setitem__(self, key, item)
                 
    def listDirectory(directory, fileExtList):
        "获取directory目录下的所有fileExtList格式的文件，fileExtList是一个列表，可以有多种格式"
        fileList = [os.path.normcase(f)
            for f in os.listdir(directory)] # 列出所有 directory的文件
        fileList = [os.path.join(directory, f)
            for f in fileList
            # 过滤文件，满足fileExtList内的一种格式。os.path.splitext将文件分成文件名和扩展名
            if os.path.splitext(f)[1] in fileExtList]
             
        # sys.modules[FileInfo.__module__] 获取FileInfo.__module__模块，其中FileInfo.__module__在此会是 main，
        # 如果被别的模块调用的话就不是了，这是为什么不直接用“main”
        def getFileInfoClass(filename, module=sys.modules[FileInfo.__module__]):
            "定义一个函数，获取文件的信息"
             # 获取需要用来解析的类，如果是mp3文件结果为MP3FileInfo,其他为FileInfo
            subclass = "%sFileInfo" % os.path.splitext(filename)[1].upper()[1:]
            # 返回一个类，注意，返回的是一个“类”。使用getattr获取moudle模块中的subclass类
            return hasattr(module, subclass) and getattr(module, subclass) or FileInfo
        # 注意，这句话可能比较难理解， getFileInfoClass(f) (f)为什么会有两个(f)呢，上面已经说过getFileInfoClass(f)
        # 根据文件名返回一个解析类，这儿是返回就是MP3FileInfo,而第二个(f)就表示对这个类以f初始化MP3FileInfo(f)
        return [getFileInfoClass(f) (f) for f in fileList]
     
    if __name__ == "__main__": # main函数，在别的模块中不会允许这里面的代码了
        for info in listDirectory("E:\\Music", [".mp3"]): # 循环获取E:\\Music文件夹中所有的mp3文件的信息
            # 由于MP3FileInfo继承自FileInfo,FileInfo继承自UserDict,这个的items()就是获取key-value集合。
            # 使用"%s=%s"格式化输出，使用"\n".join将所有信息以换行连接。
            print "\n".join(["%s=%s" % (k, v) for k, v in info.items()])
            print # 每一个文件之后，输出一个空行

结果为：  

album=What Are Words - Single

comment=pythontab

name=E:\Music\chris medina - what_are_words.mp3

title=What Are Words

artist=Chris Medina

year=2011

genre=13



album=After the Wedding

comment=pythontab

name=E:\Music\two fathers.mp3

title=Two Fathers

artist=pythontab

year=2010

genre=255

  

注意：逻辑比较多，代码不算少，不懂的多看注释

  

