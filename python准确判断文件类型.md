# python准确判断文件类型

判断文件类型在开发中非常常见的需求，怎样才能准确的判断文件类型呢？首先大家想到的是文件的后缀，但是非常遗憾的是这种方法是非常不靠谱的，因为文件的后缀是可以随
意更改的，而大家都知道后缀在linux系统下是没有这个概念的，所以仅靠判断后缀无法准确判断一个文件的类型。还有第二种方法是判断文件的头，每种文件在文件的头中
会标识这种文件的类型，下面我们来看看如何用python来判断文件的类型。

python通过文件头判断文件类型的方法：

    
    
    #! /usr/bin/python
    # pythontab提醒您注意中文编码问题，指定编码为utf-8
    # -*- coding: utf-8 -*- 
    import struct
    # 支持文件类型 
    # 用16进制字符串的目的是可以知道文件头是多少字节 
    # 各种文件头的长度不一样，少则2字符，长则8字符 
    def typeList(): 
      return { 
        "FFD8FF": "JPEG", 
        "89504E47": "PNG"} 
      
    # 字节码转16进制字符串 
    def bytes2hex(bytes): 
      num = len(bytes) 
      hexstr = u"" 
      for i in range(num): 
        t = u"%x" % bytes[i] 
        if len(t) % 2: 
          hexstr += u"0"
        hexstr += t 
      return hexstr.upper() 
      
    # 获取文件类型 
    def filetype(filename): 
      binfile = open(filename, 'rb') # 必需二制字读取 
      tl = typeList() 
      ftype = 'unknown'
      for hcode in tl.keys(): 
        numOfBytes = len(hcode) / 2 # 需要读多少字节 
        binfile.seek(0) # 每次读取都要回到文件头，不然会一直往后读取 
        hbytes = struct.unpack_from("B"*numOfBytes, binfile.read(numOfBytes)) # 一个 "B"表示一个字节 
        f_hcode = bytes2hex(hbytes) 
        if f_hcode == hcode: 
          ftype = tl[hcode] 
          break
      binfile.close() 
      return ftype 
      
    if __name__ == '__main__': 
      print filetype('./test.jpg')



常见文件格式的文件头

    
    
    文件格式 文件头(十六进制)
    JPEG (jpg) FFD8FF
    PNG (png) 89504E47
    GIF (gif) 47494638
    TIFF (tif) 49492A00
    Windows Bitmap (bmp) 424D
    CAD (dwg) 41433130
    Adobe Photoshop (psd) 38425053
    Rich Text Format (rtf) 7B5C727466
    XML (xml) 3C3F786D6C
    HTML (html) 68746D6C3E
    Email [thorough only] (eml) 44656C69766572792D646174653A
    Outlook Express (dbx) CFAD12FEC5FD746F
    Outlook (pst) 2142444E
    MS Word/Excel (xls.or.doc) D0CF11E0
    MS Access (mdb) 5374616E64617264204A

  

