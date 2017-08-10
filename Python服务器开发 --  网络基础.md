# Python服务器开发 -- 网络基础

网络由下往上分为物理层、数据链路层、网络层、传输层、会话层、表示层和应用层。

HTTP是高层协议，而TCP/IP是个协议集，包过许多的子协议。包括：传输层的
FTP，UDP，TCP协议等，网络层的ip协议等，高层协议如HTTP,telnet协议等，HTTP是TCP/IP的一个子协议。

socket是对TCP/IP协议的封装和应用（程序员层面上）。也可以说，TPC/IP协议是传输层协议，主要解决数据如何在网络中传输，而HTTP是应用层协议，
主要解决如何包装数据。

我们在传输数据时，可以只使用（传输层）TCP/IP协议，但是那样的话，如果没有应用层，便无法识别数据内容，如果想要使传输的数据有意义，则必须使用到应用层协议
，应用层协议有很多，比如HTTP、FTP、TELNET等，也可以自己定义应用层协议。WEB使用HTTP协议作应用层协议，以封装HTTP文本信息，然后使用TC
P/IP做传输层协议将它发到网络上。

而我们平时说的最多的socket是什么呢，实际上socket是对TCP/IP协议的封装，Socket本身并不是协议，而是一个调用接口（API），通过Sock
et，我们才能使用TCP/IP协议。实际上，Socket跟TCP/IP协议没有必然的联系。Socket编程接口在设计的时候，就希望也能适应其他的网络协议。所
以说，Socket的出现只是使得程序员更方便地使用TCP/IP协议栈而已，是对TCP/IP协议的抽象，从而形成了我们知道的一些最基本的函数接口，比如crea
te、listen、connect、accept、send、read和write等等。

TCP/IP只是一个协议栈，就像操作系统的运行机制一样，必须要具体实现，同时还要提供对外的操作接口。这个就像操作系统会提供标准的编程接口，比如win32编程
接口一样，TCP/IP也要提供可供程序员做网络开发所用的接口，这就是Socket编程接口。

有个比较形象的描述：HTTP是轿车，提供了封装或者显示数据的具体形式；Socket是发动机，提供了网络通信的能力。

实际上，传输层的TCP是基于网络层的IP协议的，而应用层的HTTP协议又是基于传输层的TCP协议的，而Socket本身不算是协议，就像上面所说，它只是提供了
一个针对TCP或者UDP编程的接口。

利用Socket建立网络连接的步骤：

建立Socket连接至少需要一对套接字，其中一个运行于客户端，称为ClientSocket ，另一个运行于服务器端，称为ServerSocket 。

套接字之间的连接过程分为三个步骤：服务器监听，客户端请求，连接确认。

1。服务器监听：服务器端套接字并不定位具体的客户端套接字，而是处于等待连接的状态，实时监控网络状态，等待客户端的连接请求。

2。客户端请求：指客户端的套接字提出连接请求，要连接的目标是服务器端的套接字。为此，客户端的套接字必须首先描述它要连接的服务器的套接字，指出服务器端套接字的
地址和端口号，然后就向服务器端套接字提出连接请求。

3。连接确认：当服务器端套接字监听到或者说接收到客户端套接字的连接请求时，就响应客户端套接字的请求，建立一个新的线程，把服务器端套接字的描述发给客户端，一旦
客户端确认了此描述，双方就正式建立连接。而服务器端套接字继续处于监听状态，继续接收其他客户端套接字的连接请求。

HTTP链接的特点

HTTP协议即超文本传送协议(Hypertext Transfer Protocol
)，是Web联网的基础，也是手机联网常用的协议之一，HTTP协议是建立在TCP协议之上的一种应用。

HTTP连接最显著的特点是客户端发送的每次请求都需要服务器回送响应，在请求结束后，会主动释放连接。从建立连接到关闭连接的过程称为“一次连接”。

The Internet Protocol（协议）

IP就是一个32位无符号整数。IP地址通过DNS (Domain Name System) 数据库映射到域名

    
    
    #!/usr/bin/env python
    # Foundations of Python Network Programming - Chapter 1 - getname.py
    import socket
    hostname = 'google.com'
    addr = socket.gethostbyname(hostname)
    print 'The address of', hostname, 'is', addr
    # The address of google.com is 173.194.72.113

  

Python网络编程：

Python提供了访问底层操作系统Socket接口的全部方法，还提供了一组加密和认证通信的服务，SSL／TLS。

Sockets其实是一个文件描述符，不同于不同于本地文件，它连接了网络上的一个文件。

1、创建一个UDP 本地连接：

    
    
    #!/usr/bin/env python
    import socket, sys
    s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    MAX = 65535
    PORT = 1060
    if sys.argv[1:] == ['server']:
        s.bind(('127.0.0.1', PORT))
        print 'Listening at', s.getsockname()
        while True:
            data, address = s.recvfrom(MAX)
            print 'The client at', address, 'says', repr(data)              
    　　　　 s.sendto('Your data was %d bytes' % len(data), address)
    elif sys.argv[1:] == ['client']:
            print 'Address before sending:', s.getsockname()
            s.sendto('This is my message', ('127.0.0.1', PORT))
            print 'Address after sending', s.getsockname()
            data, address = s.recvfrom(MAX) # overly promiscuous - see text!
            print 'The server', address, 'says', repr(data)
    else:
        print >>sys.stderr, 'usage: udp_local.py server|client'

  

运行这段代码：

    
    
    python filename.py server
    ＃Listening at ('127.0.0.1', 1060)
    ＃Address before sending: ('0.0.0.0', 0)
    ＃Address after sending ('0.0.0.0', 62892)
    ＃The server ('127.0.0.1', 1060) says 'Your data was 18 bytes'
    python filename.py client
    ＃The client at ('127.0.0.1', 62892) says 'This is my message'

  

2、创建远程连接并验证收到的信息：

    
    
    import random, socket, sys
    s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    MAX = 65535
    PORT = 1060
    if 2 <= len(sys.argv) <= 3 and sys.argv[1] == 'server':
        interface = sys.argv[2] if len(sys.argv) > 2 else ''
        s.bind((interface, PORT))
        print 'Listening at', s.getsockname()
        while True:
            data, address = s.recvfrom(MAX)
            if random.randint(0, 1):
                print 'The client at', address, 'says:', repr(data)
                s.sendto('Your data was %d bytes' % len(data), address)
            else:
                print 'Pretending to drop packet from', address
    elif len(sys.argv) == 3 and sys.argv[1] == 'client':
        hostname = sys.argv[2]
        s.connect((hostname, PORT))
        print 'Client socket name is', s.getsockname()
        delay = 0.1
        while True:
            s.send('This is another message')
            print 'Waiting up to', delay, 'seconds for a reply'
            s.settimeout(delay)
            try:
                data = s.recv(MAX)
            except socket.timeout:
                delay *= 2  # wait even longer for the next request
                if delay > 2.0:
                    raise RuntimeError('I think the server is down')
            else:
                break   # we are done, and can stop looping
                 
        print 'The server says', repr(data)
    else:
        print >>sys.stderr, 'usage: udp_remote.py server [ <interface> ]'
        print >>sys.stderr, '   or: udp_remote.py client <host>'
        sys.exit(2)

这里的s.connect((hostname, PORT))方法，可以让我们不用每次都调用s.sendto('This is my message',
('127.0.0.1', PORT))。直接调用

s.send('This is another message')。

  

