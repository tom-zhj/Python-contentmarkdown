# python移除系统多余大文件

文件多了乱放， 突然有一天发现硬盘空间不够了，
于是写了个python脚本搜索所有大于10MB的文件，看看这些大文件有没有重复的副本，如果有，全部列出，以便手工删除

使用方式 加一个指定目录的参数

比如python redundant_remover.py /tmp

主要用到了stat模块，os、sys系统模块

    
    
    import os, sys
    #引入统计模块
    from stat import *
    BIG_FILE_THRESHOLD = 10000000L
    dict1 = {}    # filesize 做 key, filename 做 value
    dict2 = {}     # filename 做 key, filesize 做 value
    def treewalk(path):
        try:
            for i in os.listdir(path):
                mode = os.stat(path+"/"+i).st_mode
                if S_ISDIR(mode) <> True:
                    filename = path+"/"+i
                    filesize = os.stat(filename).st_size
                    if filesize > BIG_FILE_THRESHOLD:
                        if filesize in dict1:                       
                            dict2[filename] = filesize
                            dict2[dict1[filesize]]=filesize
                        else:
                            dict1[filesize] = filename                  
                else:
                    treewalk(path+"/"+i)
        except WindowsError:
            pass
    def printdict(finaldict):
        for i_size in finaldict.values():
            print i_size
            for j_name in finaldict.keys():
                if finaldict[j_name] == i_size:
                    print j_name
            print "\n"
    if __name__=="__main__":
        treewalk(sys.argv[1])
        printdict(dict2)

  

  

