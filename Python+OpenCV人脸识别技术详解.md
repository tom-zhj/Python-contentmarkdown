# Python+OpenCV人脸识别技术详解

总在科幻电影里看到人脸识别，现在我们也可以编程来实现啦。哈哈~~  
OpenCV是Intel‍‍®‍‍开源计算机视觉库。它由一系列 C 函数和少量 C++ 类构成，实现了图像处理和计算机视觉方面的很多通用算法。  
OpenCV 拥有包括 300 多个C函数的跨平台的中、高层 API。它不依赖于其它的外部库--
尽管也可以使用某些外部库。它还提供了Python、Ruby、MATLAB等语言的接口，实现了图像处理和计算机视觉方  
  
面的很多通用算法。  
  
所以总体来说OpenCV的人脸检测功能在是很不错的。  
  
效果图如下：  
  
![](http://www.pythontab.com/uploadfile/2017/0321/20170321114220979.png)

下面我们就用python + OpenCV实现人脸识别。  
  
开发运行环境：  
Centos5.5  
OpenCV  
python2.7  
PIL  
  
下面上代码：  
  

    
    
    #!/usr/bin/python
    # -*- coding: UTF-8 -*-
     
    # face_detect.py
     
    # Face Detection using OpenCV. Based on sample code from:
    # http://www.pythontab.com
     
    # Usage: python face_detect.py
     
    import sys, os
    #引入opencv库中的相应组件
    from opencv.cv import *
    from opencv.highgui import *
    #引入PIL库
    from PIL import Image, ImageDraw
    from math import sqrt
     
    def detectObjects(image):
        #首先把图片转换为灰度模式，以便找到人脸位置
        grayscale = cvCreateImage(cvSize(image.width, image.height), 8, 1)
        cvCvtColor(image, grayscale, CV_BGR2GRAY)
     
        storage = cvCreateMemStorage(0)
        cvClearMemStorage(storage)
        cvEqualizeHist(grayscale, grayscale)
     
        cascade = cvLoadHaarClassifierCascade(
            \'/usr/share/opencv/haarcascades/haarcascade_frontalface_default.xml\',
            cvSize(1,1))
        faces = cvHaarDetectObjects(grayscale, cascade, storage, 1.1, 2,
            CV_HAAR_DO_CANNY_PRUNING, cvSize(20,20))
     
        result = []
        for f in faces:
            result.append((f.x, f.y, f.x+f.width, f.y+f.height))
     
        return result
     
    def grayscale(r, g, b):
        return int(r * .3 + g * .59 + b * .11)
     
    def process(infile, outfile):
     
        image = cvLoadImage(infile);
        if image:
            faces = detectObjects(image)
     
        im = Image.open(infile)
     
        if faces:
            draw = ImageDraw.Draw(im)
            for f in faces:
                draw.rectangle(f, outline=(255, 0, 255))
     
            im.save(outfile, "JPEG", quality=100)
        else:
            print "Error: cannot detect faces on %s" % infile
     
    if __name__ == "__main__":
        process(\'input.jpg\', \'output.jpg\')

代码到此结束，上面的例子看不懂，没关系，因为我们大量使用了库里面的函数和方法，如果看不懂，我们可以去网上查或者使用手册，只要借助这些看懂这段代码就ok，重要
的是掌握其中的人脸识别实现思想  

