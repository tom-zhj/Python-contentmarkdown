# python 开发简单的聊天工具

python 太强大了，以至于它什么都可以做，哈哈，开个玩笑。但是今天要讲的真的是一个非常神奇的应用。

使用python写一个聊天工具

其实大家平时用的QQ类似的聊天工具，也是使用socket进行聊天，只是它还包含了更加复杂的功能。基本原理是一样的。

python实现聊天功能，主要用到了socket模块。下面直接上实例吧

  

server端

    
    
    import socket
    s=socket.socket()
    #建立socket链接
    s.bind(('127.0.0.1',8000))
    #监听连接请求，其中的1 ，是指监听一个
    s.listen(1)
    #进行循环，一直监听client发来的消息
    while 1:
    #获取链接IP和端口
        conn,addr=s.accept()
        print '['+addr[0]+':'+str(addr[1])+'] send a message to me: '+conn.recv(1024)
        conn.sendall('I received a message from ['+addr[0]+':'+str(addr[1])+']')
    s.close()

  

client 端，比较简单不需要监听，不需要循环  

    
    
    import socket
    s=socket.socket()
    #链接
    s.connect(('127.0.0.1', 8000))
    #获取键盘输入
    msg = raw_input("Please input your message:")
    s.sendall(msg)
    print s.recv(1024)
    s.close()

  

很简单吧，哈哈，大家可以在这个代码的基础上，优化创造出更强的功能

  

