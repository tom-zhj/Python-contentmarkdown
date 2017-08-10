# python用两种方法实现url短连接

几乎所有的微薄都提供了缩短网址的服务，其原理就是将一个url地址按照一定的算法生成一段字符串，然后加在一个短域名后面边成了一个新的url地址，数据库中会存放
这个短地址和原始的地址，当用户点击这个新的短地址后，短地址服务会根据短域名后面的几个字符串从数据库中读出原来的地址然后页面进行跳转 。

比如新浪微薄中的url 是 http://t.cn/xxxxxxx  t.cn是其域名 ，其后面跟着的是7位算出来的字符串。

## 方法一：使用哈希库自定义算法  

因为文本中显示太长的url会比较乱,或者采用省略显示的方式,或者采用短url的方式.

为了同时方便统计点击数以及进行内容过滤.实现了一个生成短url值的方法.

为了防止你的hash值被破解,可以在生成md5值的时候加入你自己的salt.

这样即便直到你的code_map也不能破解到原始url了.

为了让结果更加随机,把每次循环没有使用的第二个bit保存到e里面.这样可以让结果冲突率更小.

    
    
    #引入哈希库
    import hashlib  
          
    def get_md5(s):  
        s = s.encode('utf8') if isinstance(s, unicode) else s  
        m = hashlib.md5()  
        m.update(s)  
        return m.hexdigest()  
          
    code_map = (  
               'a' , 'b' , 'c' , 'd' , 'e' , 'f' , 'g' , 'h' ,  
               'i' , 'j' , 'k' , 'l' , 'm' , 'n' , 'o' , 'p' ,  
               'q' , 'r' , 's' , 't' , 'u' , 'v' , 'w' , 'x' ,  
               'y' , 'z' , '0' , '1' , '2' , '3' , '4' , '5' ,  
               '6' , '7' , '8' , '9' , 'A' , 'B' , 'C' , 'D' ,  
               'E' , 'F' , 'G' , 'H' , 'I' , 'J' , 'K' , 'L' ,  
               'M' , 'N' , 'O' , 'P' , 'Q' , 'R' , 'S' , 'T' ,  
               'U' , 'V' , 'W' , 'X' , 'Y' , 'Z'
                )  
          
          
    def get_hash_key(long_url):  
        hkeys = []  
        hex = get_md5(long_url)  
        for i in xrange(0, 4):  
            n = int(hex[i*8:(i+1)*8], 16)  
            v = []  
            e = 0
            for j in xrange(0, 5):  
                x = 0x0000003D & n  
                e |= ((0x00000002 & n ) >> 1) << j  
                v.insert(0, code_map[x])  
                n = n >> 6
            e |= n << 5
            v.insert(0, code_map[e & 0x0000003D])  
            hkeys.append(''.join(v))  
        return hkeys  
          
    if __name__ == '__main__':  
        print get_hash_key('http://www.pythontab.com')

## 方法二：使用libsurl库

libsurl 是一个用来生成短URL的C和Python库，支持 bit.ly 和 tinyurl 等短url 服务网站。  

