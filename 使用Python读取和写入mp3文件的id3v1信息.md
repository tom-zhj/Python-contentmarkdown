# 使用Python读取和写入mp3文件的id3v1信息

1.起因

一直以来疯迷“冬吴相对论”，为了整理下载他的MP3花了不少功夫，今天突然发现将电脑中的mp3导入到itunes后，文件名竟然不识别了。#_*
itunes自动识别了mp3的信息内容。多次一举么，文件名挺好。事实如此，让我深感不完美。一定要将文件名也写如MP3信息中区。

网上一搜，一大把的python代码，都是用了eyeD3这个组件包。照着例子简单搞了两下就出来一个版本，运行发现latin_1啥的编码问题。OK把它的tag和
id3还有frames包中的编码统统改成GBK就能解决了。但是又发现，如果文件原本没有id3v1时，获取title就直接报错了。找了两下没有发现有人提这个问
题。看来只能自己动手了。那就完全不用eyeD3包了。因为id3v1确实很简单。

2.分析

百度就有说，我想写的这些信息可保存于mp3文件的尾部。

ID3V1比较简单，它是存放在MP3文件的末尾，用16进制的编辑器打开一个MP3文件，查看其末尾的128个顺序存放字节，数据结构定义如下：

char Header[3]; /标签头必须是"TAG"否则认为没有标签/

char Title[30]; /标题/

char Artist[30]; /作者/

char Album[30]; /专集/

char Year[4]; /出品年代/

char Comment[30]; /备注/

char Genre; /类型/

ID3V1的各项信息都是顺序存放，没有任何标识将其分开，比如标题信息不足30个字节，则使用'\0'补足，否则将造成信息错误。

3.解决

还好，文件结构不复杂，处理起来就相对简单。思路很简单，读取mp3文件的尾部128字节，判断一下有米有TAG，有了就把最后的128节用我们自己的信息替换掉，没
有就补充128字节上去。

4.代码

最好的文档就是源码，当然我回写注释的。没有依赖eyeD3这样的包，纯手工写法。

    
    
    #encoding=utf8
    __author__ ='pcode@qq.com'import os
    importstructdefGetFiles(path):"""
        读取指定目录的文件
        """FileDic=[]
        files=os.listdir(path)for f in files:
            f=f[:-4]FileDic.append(f)returnFileDic,files
    def_GetLast128K(path,file):
        ff1=open(os.path.join(path,file),"rb")
        ff1.seek(-128,2)
        id3v1data=ff1.read()
        ff1.close()return id3v1data
    def_GetAllBinData(path,file):
        ff1=open(os.path.join(path,file),"rb")
        data=ff1.read()
        ff1.close()return data
    defSetTag(path,file,title,artist,album,year,comment,genre):"""
        设置mp3的ID3 v1中的部分参数
        char Header[3]; /*标签头必须是"TAG"否则认为没有标签*/
        char Title[30]; /*标题*/
        char Artist[30]; /*作者*/
        char Album[30]; /*专集*/
        char Year[4]; /*出品年代*/
        char Comment[30]; /*备注*/
        char Genre; /*类型*/
        mp3文件尾部128字节为id3v1的数据，如果有数据则读取修改，无数据则补充
        """
        header='TAG'#组合出最后128K的id3V1的数据内容
        str =struct.pack('3s30s30s30s4s30ss',header,title,artist,album,year,comment,genre)#获取原始全部数据
        data=_GetAllBinData(path,file)#获取末尾的128字节数据
        id3v1data=_GetLast128K(path,file)#打开原文件准备写入
        ff=open(os.path.join(path,file),"wb")try:#判断是否有id3v1数据if id3v1data[0:3]!=header:#倒数128字节不是以TAG开头的说明没有#按照id3v1的结构补充上去
                ff.write(data+str)else:#有的情况下要换一下
                ff.write(data[0:-128]+str)
            ff.close()print"OK"+title
        except:
            ff.write(data)print"Error "+title
        finally:if ff :ff.close()if __name__=="__main__":#我存放mp3文件的目录
        path=u"K:\\reading\\阅读\\相对论"#获取到文件名和文件全名
        names,files=GetFiles(path)#苦力代码for i in range(len(files)):#注意编码解码
            title=names[i].encode('gbk')
            artist=u'作者'.encode('gbk')
            album=u'相对论'.encode('gbk')
            year=''
            comment=''
            genre=''#调用函数处理SetTag(path,files[i],title,artist,album,year,comment,genre)

5.后续

使用了以后id3v1的信息全部按文件名改好了，其中的SetTag函数也可以迁移到别的程序里用来改id3v1的信息。但是写文件那里，无论是否有TAG都得重写全
部文件内容。效率一般般。速度没有eyeD3这种组件快。但那时eyeD3不能支持中文，而且文件本来没id3v1信息时会出错，自己的就放心多了。 bingo
收工。

  

