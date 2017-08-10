# 使用python判断流媒体mp3格式

项目中使用mp3格式进行音效播放，遇到一个mp3文件在程序中死活播不出声音，最后发现它是wav格式的文件，却以mp3结尾。要对资源进行mp3格式判断，那么如
何判断呢，用.mp3后缀肯定不靠谱，我们知道扩展名是可以任意修改的，得从编码格式判断，方法如下：

  

  1. ### mp3编码

MP3文件是一种流媒体文件格式，所以没有文件头。像AVI、WAV这种有文件头的格式，很好判断，他们都是RIFF开头的，只要进行RIFF字符串对比，就可以查出
是否是AVI、WAV，而mp3就只能分析编码格式了。这里大概说mp3编码规则一下，详细的可用参考这篇文章

MP3 文件大体分为三部分：TAG_V2(ID3V2)，音频数据，TAG_V1(ID3V1)

a). ID3V2 在文件开始的位置，以ID3开头，包含了作者，作曲，专辑等信息，长度不固定，扩展了ID3V1 的信息量，非必需

b). 一系列的音频数据的帧，在文件的中间位置，个数由文件大小和帧长决定；每个帧都以FFF开头，的长度可能不固定，也可能固定，由位率bitrate决定；每个
帧又分为帧头和数据实体两部分；帧头记录了mp3 的位率，采样率，版本等信息，每个帧之间相互独立 。

c). ID3V1在文件结尾的位置，以TAG开头，包含了作者，作曲，专辑等信息，长度为128Byte，非必须。

ID3V2

包含了作者，作曲，专辑等信息，长度不固定，扩展了ID3V1的信息量。

Frame

.

.

.

Frame

一系列的帧，个数由文件大小和帧长决定

每个FRAME的长度可能不固定，也可能固定，由位率bitrate决定

每个FRAME又分为帧头和数据实体两部分

帧头记录了mp3的位率，采样率，版本等信息，每个帧之间相互独立。

ID3V1

包含了作者，作曲，专辑等信息，长度为128BYTE。

 也就是说，根据TAG_V2(ID3V2)，音频数据，TAG_V1(ID3V1)三结构中的开头信息，便可以判断出是不是mp3编码的文件。

  

### 2.python代码

    
    
    # coding: utf-8
    import os
    #mp3filePath是否是mp3格式的
    def isMp3Format(mp3filePath):
     #读取文件内字符串
     f = open(mp3filePath, "r");
     fileStr = f.read();
     f.close();
     head3Str = fileStr[:3];
     #判断开头是不是ID3
     if head3Str == "ID3":
      return True;
     #判断结尾有没有TAG
     last32Str = fileStr[-32:];
     if last32Str[:3] == "TAG":
      return True;
     #判断第一帧是不是FFF开头, 转成数字
     # fixme 应该循环遍历每个帧头，这样才能100%判断是不是mp3
     ascii = ord(fileStr[:1]);
     if ascii == 255:
      return True;
     return False;
    #遍历folderPath看看是不是都是mp3格式的，
    #是就true，不是就是false, 并返回是mp3的list，不是MP3的list
    def isMp3FolderTraverse(folderPath):
     mp3List = [];
     notMp3List = [];
     isAllMpFormat = True;
     for dirpath, dirnames, filenames in os.walk(folderPath):
      for filename in filenames:
       path = dirpath + os.sep + filename;
       isMp3 = isMp3Format(path);
       #判断是不是mp3结尾的 并且 是mp3格式的
       if isMp3 == False and str.endswith(path, ".mp3") == True:
        # print("--warning: file " + path + " is not mp3 format!--");
        notMp3List.append(path);
        isAllMpFormat = False;
       else:
        mp3List.append(path);
     return isAllMpFormat, mp3List, notMp3List;
    if __name__ == '__main__':
     isMp3Format("s_com_click1.mp3");
     isAllMp3, mp3List, notMp3List = isMp3FolderTraverse("sound");
     print isAllMp3;
     print mp3List;
     print notMp3List;

  

