# python网络编程——IO多路复用之epoll

### 什么是epoll

epoll是什么？在linux的网络编程中，很长的时间都在使用select来做事件触发。在linux新的内核中，有了一种替换它的机制，就是epoll。当然，
这不是2.6内核才有的，它是在2.5.44内核中被引进的(epoll(4) is a new API introduced in Linux kernel
2.5.44)，它几乎具备了之前所说的一切优点，被公认为Linux2.6下性能最好的多路复用I/O就绪通知方法。

相比于select，epoll最大的好处在于它不会随着监听fd数目的增长而降低效率。因为在内核中的select实现中，它是采用轮询来处理的，轮询的fd数目越
多，自然耗时越多。

### epoll工作原理

epoll同样只告知那些就绪的文件描述符，而且当我们调用epoll_wait()获得就绪文件描述符时，返回的不是实际的描述符，而是一个代表就绪描述符数量的值
，你只需要去epoll指定的一个数组中依次取得相应数量的文件描述符即可，这里也使用了内存映射（mmap）技术，这样便彻底省掉了这些文件描述符在系统调用时复制
的开销。



另一个本质的改进在于epoll采用基于事件的就绪通知方式。在select/poll中，进程只有在调用一定的方法后，内核才对所有监视的文件描述符进行扫描，而e
poll事先通过epoll_ctl()来注册一个文件描述符，一旦基于某个文件描述符就绪时，内核会采用类似callback的回调机制，迅速激活这个文件描述符，
当进程调用epoll_wait()时便得到通知。

  

从以上可知，epoll是对select、poll模型的改进，提高了网络编程的性能，广泛应用于大规模并发请求的C/S架构中。

### python中的epoll

#### 1、触发方式：

     边缘触发/水平触发，只适用于Unix/Linux操作系统

#### 2、原理图

![](http://www.pythontab.com/uploadfile/2016/1010/20161010104745752.png)

#### 3、一般步骤

Create an epoll object——创建1个epoll对象

Tell the epoll object to monitor specific events on specific
sockets——告诉epoll对象，在指定的socket上监听指定的事件

Ask the epoll object which sockets may have had the specified event since the
last query——询问epoll对象，从上次查询以来，哪些socket发生了哪些指定的事件

Perform some action on those sockets——在这些socket上执行一些操作

Tell the epoll object to modify the list of sockets and/or events to
monitor——告诉epoll对象，修改socket列表和（或）事件，并监控

Repeat steps 3 through 5 until finished——重复步骤3-5，直到完成

Destroy the epoll object——销毁epoll对象

#### 4、相关用法

import select 导入select模块

epoll = select.epoll()创建一个epoll对象

epoll.register(文件句柄,事件类型)注册要监控的文件句柄和事件

事件类型:

　　select.EPOLLIN    可读事件

　　select.EPOLLOUT   可写事件

　　select.EPOLLERR   错误事件

　　select.EPOLLHUP   客户端断开事件

epoll.unregister(文件句柄)  销毁文件句柄

epoll.poll(timeout) 当文件句柄发生变化，则会以列表的形式主动报告给用户进程,timeout

                     为超时时间，默认为-1，即一直等待直到文件句柄发生变化，如果指定为1

                     那么epoll每1秒汇报一次当前文件句柄的变化情况，如果无变化则返回空

epoll.fileno() 返回epoll的控制文件描述符(Return the epoll control file descriptor)

epoll.modfiy(fineno,event)fineno为文件描述符 event为事件类型  作用是修改文件描述符所对应的事件

epoll.fromfd(fileno)从1个指定的文件描述符创建1个epoll对象

epoll.close()   关闭epoll对象的控制文件描述符

#### 5 实例：客户端发送数据 服务端将接收的数据返回给客户端

服务端代码

    
    
    #!/usr/bin/env python
    #-*- coding:utf-8 -*-
    import socket
    import select
    import Queue
    #创建socket对象
    serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    #设置IP地址复用
    serversocket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    #ip地址和端口号
    server_address = ("127.0.0.1", 8888)
    #绑定IP地址
    serversocket.bind(server_address)
    #监听，并设置最大连接数
    serversocket.listen(10)
    print  "服务器启动成功，监听IP：" , server_address
    #服务端设置非阻塞
    serversocket.setblocking(False)  
    #超时时间
    timeout = 10
    #创建epoll事件对象，后续要监控的事件添加到其中
    epoll = select.epoll()
    #注册服务器监听fd到等待读事件集合
    epoll.register(serversocket.fileno(), select.EPOLLIN)
    #保存连接客户端消息的字典，格式为{}
    message_queues = {}
    #文件句柄到所对应对象的字典，格式为{句柄：对象}
    fd_to_socket = {serversocket.fileno():serversocket,}
    while True:
      print "等待活动连接......"
      #轮询注册的事件集合，返回值为[(文件句柄，对应的事件)，(...),....]
      events = epoll.poll(timeout)
      if not events:
         print "epoll超时无活动连接，重新轮询......"
         continue
      print "有" , len(events), "个新事件，开始处理......"
      
      for fd, event in events:
         socket = fd_to_socket[fd]
         #如果活动socket为当前服务器socket，表示有新连接
         if socket == serversocket:
                connection, address = serversocket.accept()
                print "新连接：" , address
                #新连接socket设置为非阻塞
                connection.setblocking(False)
                #注册新连接fd到待读事件集合
                epoll.register(connection.fileno(), select.EPOLLIN)
                #把新连接的文件句柄以及对象保存到字典
                fd_to_socket[connection.fileno()] = connection
                #以新连接的对象为键值，值存储在队列中，保存每个连接的信息
                message_queues[connection]  = Queue.Queue()
         #关闭事件
         elif event & select.EPOLLHUP:
            print 'client close'
            #在epoll中注销客户端的文件句柄
            epoll.unregister(fd)
            #关闭客户端的文件句柄
            fd_to_socket[fd].close()
            #在字典中删除与已关闭客户端相关的信息
            del fd_to_socket[fd]
         #可读事件
         elif event & select.EPOLLIN:
            #接收数据
            data = socket.recv(1024)
            if data:
               print "收到数据：" , data , "客户端：" , socket.getpeername()
               #将数据放入对应客户端的字典
               message_queues[socket].put(data)
               #修改读取到消息的连接到等待写事件集合(即对应客户端收到消息后，再将其fd修改并加入写事件集合)
               epoll.modify(fd, select.EPOLLOUT)
         #可写事件
         elif event & select.EPOLLOUT:
            try:
               #从字典中获取对应客户端的信息
               msg = message_queues[socket].get_nowait()
            except Queue.Empty:
               print socket.getpeername() , " queue empty"
               #修改文件句柄为读事件
               epoll.modify(fd, select.EPOLLIN)
            else :
               print "发送数据：" , data , "客户端：" , socket.getpeername()
               #发送数据
               socket.send(msg)
    #在epoll中注销服务端文件句柄
    epoll.unregister(serversocket.fileno())
    #关闭epoll
    epoll.close()
    #关闭服务器socket
    serversocket.close()

客户端代码：

    
    
    #!/usr/bin/env python
    #-*- coding:utf-8 -*-
    import socket
    #创建客户端socket对象
    clientsocket = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    #服务端IP地址和端口号元组
    server_address = ('127.0.0.1',8888)
    #客户端连接指定的IP地址和端口号
    clientsocket.connect(server_address)
    while True:
        #输入数据
        data = raw_input('please input:')
        #客户端发送数据
        clientsocket.sendall(data)
        #客户端接收数据
        server_data = clientsocket.recv(1024)
        print '客户端收到的数据：'server_data
        #关闭客户端socket
        clientsocket.close()

  

