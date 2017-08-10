# linux及windows下使用Python获取IP地址

使用Python可以用很简单的方法得到本机IP地址，不过在Windows和Linux下的方法稍有不一样的，下面就来详细介绍下：

### Windows下获得IP地址的方法

  

**方法一 使用socket模块**

使用拨号上网的话，一般都有一个本地ip和一个外网ip，使用python可以很容易的得到这两个ip
使用gethostbyname和gethostbyname_ex两个函数可以实现

    
    
    #使用socket模块
    import socket
    #得到本地ip
    localIP = socket.gethostbyname(socket.gethostname())
    print"local ip:%s "%localIP
            
    ipList = socket.gethostbyname_ex(socket.gethostname())for i in ipList:
        if i != localIP:
           print"external IP:%s"%i

  

或者

    
    
    #引入socket模块
    import socket
    myname = socket.getfqdn(socket.gethostname())
    myaddr = socket.gethostbyname(myname)

  

**方法二 使用正则表达式和urllib2模块**

该方法获取公网IP使用的是利用其他网站提供的IP检测功能，然后在使用python抓取页面，正则匹配或得。不过该方法比较准确哦

    
    
    import re,urllib2
    from subprocess import Popen, PIPE
           
    print "本机的私网IP地址为：" + re.search('\d+\.\d+\.\d+\.\d+',Popen('ipconfig', stdout=PIPE).stdout.read()).group(0)
           
    #利用其他网站提供的接口，使用urllib2获取其中的ip
    print "本机的公网IP地址为：" + re.search('\d+\.\d+\.\d+\.\d+',urllib2.urlopen("http://www.ip138.com").read()).group(0)

  

### Linux下获得IP地址的方法

  

  

上面的方法在Linux下也可以使用，除此之外，Linux下还可以用下面的方法得到本机IP地址。

  

    
    
    import socket
    import fcntl
    import struct
         
    def get_ip_address(ifname):
        skt = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        print skt
        pktString = fcntl.ioctl(skt.fileno(), 0x8915, struct.pack('256s', ifname[:15]))
        print pktString
        ipString  = socket.inet_ntoa(pktString[20:24])
        print ipString
        return ipString
         
    print get_ip_address('lo')
    print get_ip_address('eth1')

  

