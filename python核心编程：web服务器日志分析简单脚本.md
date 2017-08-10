# python核心编程：web服务器日志分析简单脚本

由于N种原因，一个分析入侵日志的任务落在了我身上，1G的日志，怎么去快速分析呢？？刺总说可以搞个脚本解析入库，再到数据库分析。。。算了，那就蛋疼了，直接码个
脚本把有问题的日志拿出来分析吧。于是就有了这个小脚本。至于怎么用就要看你自己了，哈哈，比如查到sql注入语句，然后看到IP，就可以改下脚本，用IP为特征取出
日志，分析入侵过程。速度很快哦，我那破机器，跑1G日志文件也就几秒钟的啦。

在工作中写程序完成任务是很快乐的事，也很有意思。哈哈

使用参数：seay.py E:/1.log

    
    
    #coding = utf8
    #Filename = seay.py
    import os
    import sys
     
    #特征，可以随意改，两块五一次
    _tezheng = {'union','select','file_put_contents'}
     
    def CheckFile(_path):
         
        _f = open(_path,"r")
        _All_Line = _f.readlines() 
        _f.close()
         
        _Count_Line =0
        _Len_Line = len(_All_Line)
             
        _Ex_Str = ''
     
        print('Read Over --')
         
        while _Count_Line<_Len_Line:
                _Str = _All_Line[_Count_Line]            
                for _tz_Str in _tezheng:
                    if _tz_Str in _Str: #可以加and条件，这个贵一点，5毛一次
                        _Ex_Str+=_tz_Str+_Str+'\r\n'
                _Count_Line+=1
         
        _f1 = open(_path+'.seay.txt',"w")
        _f1.write(_Ex_Str)
        _f1.close()    
        print 'Find Over--'   
     
    if len(sys.argv)==2:
        _File = sys.argv[1]
        if os.path.lexists(_File):
            CheckFile(_File)
        else:
            print('File does not exist!')
    else:
        print 'Parameter error'
        print sys.argv[0]+' FilePath'

  

最终生成一个文件为：原文件名.seay.txt在同目录下，格式为匹配的特征+日志

  

