# 得到一张图片或logo的主要颜色（颜色趋向）python版

  

在使用google或者baidu搜图的时候会发现有一个图片颜色选项，感觉非常有意思，有人可能会想这肯定是人为的去划分的，呵呵，有这种可能，但是估计人会累死，
开个玩笑，当然是通过机器识别的，海量的图片只有机器识别才能做到。

那用python能不能实现这种功能呢？答案是：能

  

利用python的PIL模块的强大的图像处理功能就可以做到，下面上代码：

    
    
    import colorsys
     
    def get_dominant_color(image):
        
    #颜色模式转换，以便输出rgb颜色值
        image = image.convert('RGBA')
        
    #生成缩略图，减少计算量，减小cpu压力
        image.thumbnail((200, 200))
        
        max_score = None
        dominant_color = None
        
        for count, (r, g, b, a) in image.getcolors(image.size[0] * image.size[1]):
            # 跳过纯黑色
            if a == 0:
                continue
            
            saturation = colorsys.rgb_to_hsv(r / 255.0, g / 255.0, b / 255.0)[1]
           
            y = min(abs(r * 2104 + g * 4130 + b * 802 + 4096 + 131072) >> 13, 235)
           
            y = (y - 16.0) / (235 - 16)
            
            # 忽略高亮色
            if y > 0.9:
                continue
            
            # Calculate the score, preferring highly saturated colors.
            # Add 0.1 to the saturation so we don't completely ignore grayscale
            # colors by multiplying the count by zero, but still give them a low
            # weight.
            score = (saturation + 0.1) * count
            
            if score > max_score:
                max_score = score
                dominant_color = (r, g, b)
        
        return dominant_color
    如何使用：
    from PIL import Image
     
    print  get_dominant_color(Image.open('logo.jpg'))

  

这样就会返回一个rgb颜色，但是这个值是很精确的范围，那我们如何实现百度图片那样的色域呢？？

其实方法很简单，r/g/b都是0-255的值，我们只要把这三个值分别划分相等的区间，然后组合，取近似值。例如：划分为0-127,和128-255，然后自由组
合，可以出现八种组合，然后从中挑出比较有代表性的颜色即可。

当然我只是举一个例子，你也可以划分的更细，那样显示的颜色就会更准确~~大家赶快试试吧

  

