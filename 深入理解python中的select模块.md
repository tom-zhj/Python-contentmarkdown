# 深入理解python中的select模块

### 简介

Python中的select模块专注于I/O多路复用，提供了select  poll
epoll三个方法(其中后两个在Linux中可用，windows仅支持select)，另外也提供了kqueue方法(freeBSD系统)

### select方法

进程指定内核监听哪些文件描述符(最多监听1024个fd)的哪些事件，当没有文件描述符事件发生时，进程被阻塞；当一个或者多个文件描述符事件发生时，进程被唤醒。

当我们调用select()时：

　　1、上下文切换转换为内核态

　　2、将fd从用户空间复制到内核空间

　　3、内核遍历所有fd，查看其对应事件是否发生

　　4、如果没发生，将进程阻塞，当设备驱动产生中断或者timeout时间后，将进程唤醒，再次进行遍历

　　5、返回遍历后的fd

　　6、将fd从内核空间复制到用户空间

fd:file descriptor 文件描述符

fd_r_list, fd_w_list, fd_e_list = select.select(rlist, wlist, xlist,
[timeout])

参数： 可接受四个参数（前三个必须）

rlist: wait until ready for reading

wlist: wait until ready for writing

xlist: wait for an “exceptional condition”

timeout: 超时时间

返回值：三个列表

select方法用来监视文件描述符(当文件描述符条件不满足时，select会阻塞)，当某个文件描述符状态改变后，会返回三个列表

    1、当参数1 序列中的fd满足“可读”条件时，则获取发生变化的fd并添加到fd_r_list中

    2、当参数2 序列中含有fd时，则将该序列中所有的fd添加到 fd_w_list中

    3、当参数3 序列中的fd发生错误时，则将该发生错误的fd添加到 fd_e_list中

    4、当超时时间为空，则select会一直阻塞，直到监听的句柄发生变化

   当超时时间 ＝ n(正整数)时，那么如果监听的句柄均无任何变化，则select会阻塞n秒，之后返回三个空列表，如果监听的句柄有变化，则直接执行。

  

### 实例：

利用select实现一个可并发的服务端

    
    
    import socket
    import select
     
    s = socket.socket()
    s.bind(('127.0.0.1',8888))
    s.listen(5)
    r_list = [s,]
    num = 0
    while True:
     rl, wl, error = select.select(r_list,[],[],10)
     num+=1
     print('counts is %s'%num)
     print("rl's length is %s"%len(rl))
     for fd in rl:
      if fd == s:
       conn, addr = fd.accept()
       r_list.append(conn)
       msg = conn.recv(200)
       conn.sendall(('first----%s'%conn.fileno()).encode())
      else:
       try:
        msg = fd.recv(200)
        fd.sendall('second'.encode())
       except ConnectionAbortedError:
        r_list.remove(fd)
     
    s.close()

  

    
    
    import socket
     
    flag = 1
    s = socket.socket()
    s.connect(('127.0.0.1',8888))
    while flag:
     input_msg = input('input>>>')
     if input_msg == '0':
      break
     s.sendall(input_msg.encode())
     msg = s.recv(1024)
     print(msg.decode())
     
    s.close()

在服务端我们可以看到，我们需要不停的调用select， 这就意味着：

　　1  当文件描述符过多时，文件描述符在用户空间与内核空间进行copy会很费时

　　2  当文件描述符过多时，内核对文件描述符的遍历也很浪费时间

　　3  select最大仅仅支持1024个文件描述符

poll与select相差不大，本文不作介绍

### epoll方法：

epoll很好的改进了select：

　　1、epoll的解决方案在epoll_ctl函数中。每次注册新的事件到epoll句柄中时，会把所有的fd拷贝进内核，而不是在epoll_wait的时候重
复拷贝。epoll保证了每个fd在整个过程中只会拷贝一次。

　　2、epoll会在epoll_ctl时把指定的fd遍历一遍（这一遍必不可少）并为每个fd指定一个回调函数，当设备就绪，唤醒等待队列上的等待者时，就会调用
这个回调函数，而这个回调函数会把就绪的fd加入一个就绪链表。epoll_wait的工作实际上就是在这个就绪链表中查看有没有就绪的fd

　　3、epoll对文件描述符没有额外限制

  

select.epoll(sizehint=-1, flags=0) 创建epoll对象



epoll.close()

Close the control file descriptor of the epoll object.关闭epoll对象的文件描述符



epoll.closed

True if the epoll object is closed.检测epoll对象是否关闭



epoll.fileno()

Return the file descriptor number of the control fd.返回epoll对象的文件描述符



epoll.fromfd(fd)

Create an epoll object from a given file descriptor.根据指定的fd创建epoll对象



epoll.register(fd[, eventmask])

Register a fd descriptor with the epoll object.向epoll对象中注册fd和对应的事件



epoll.modify(fd, eventmask)

Modify a registered file descriptor.修改fd的事件



epoll.unregister(fd)

Remove a registered file descriptor from the epoll object.取消注册



epoll.poll(timeout=-1, maxevents=-1)

Wait for events. timeout in seconds
(float)阻塞，直到注册的fd事件发生,会返回一个dict，格式为：{(fd1,event1),(fd2,event2),……(fdn,eventn)}

事件：

  

EPOLLIN Available for read 可读 状态符为1

EPOLLOUT Available for write 可写 状态符为4

EPOLLPRI Urgent data for read

EPOLLERR Error condition happened on the assoc. fd 发生错误 状态符为8

EPOLLHUP Hang up happened on the assoc. fd 挂起状态

EPOLLET Set Edge Trigger behavior, the default is Level Trigger behavior
默认为水平触发，设置该事件后则边缘触发

EPOLLONESHOT Set one-shot behavior. After one event is pulled out, the fd is
internally disabled

EPOLLRDNORM Equivalent to EPOLLIN

EPOLLRDBAND Priority data band can be read.

EPOLLWRNORM Equivalent to EPOLLOUT

EPOLLWRBAND Priority data may be written.

EPOLLMSG Ignored.

水平触发和边缘触发：

Level_triggered(水平触发，有时也称条件触发)：当被监控的文件描述符上有可读写事件发生时，epoll.poll()会通知处理程序去读写。如果这
次没有把数据一次性全部读写完(如读写缓冲区太小)，那么下次调用 epoll.poll()时，它还会通知你在上没读写完的文件描述符上继续读写，当然如果你一直不
去读写，它会一直通知你！！！如果系统中有大量你不需要读写的就绪文件描述符，而它们每次都会返回，这样会大大降低处理程序检索自己关心的就绪文件描述符的效率！！！
优点很明显：稳定可靠

Edge_triggered(边缘触发，有时也称状态触发)：当被监控的文件描述符上有可读写事件发生时，epoll.poll()会通知处理程序去读写。如果这次
没有把数据全部读写完(如读写缓冲区太小)，那么下次调用epoll.poll()时，它不会通知你，也就是它只会通知你一次，直到该文件描述符上出现第二次可读写事
件才会通知你！！！这种模式比水平触发效率高，系统不会充斥大量你不关心的就绪文件描述符！！！缺点：某些条件下不可靠

### epoll实例：

    
    
    
    import socket
    import select
     
    s = socket.socket()
    s.bind(('127.0.0.1',8888))
    s.listen(5)
    epoll_obj = select.epoll()
    epoll_obj.register(s,select.EPOLLIN)
    connections = {}
    while True:
     events = epoll_obj.poll()
     for fd, event in events:
      print(fd,event)
      if fd == s.fileno():
       conn, addr = s.accept()
       connections[conn.fileno()] = conn
       epoll_obj.register(conn,select.EPOLLIN)
       msg = conn.recv(200)
       conn.sendall('ok'.encode())
      else:
       try:
        fd_obj = connections[fd]
        msg = fd_obj.recv(200)
        fd_obj.sendall('ok'.encode())
       except BrokenPipeError:
        epoll_obj.unregister(fd)
        connections[fd].close()
        del connections[fd]
     
    s.close()
    epoll_obj.close()
    
    
    import socket
     
    flag = 1
    s = socket.socket()
    s.connect(('127.0.0.1',8888))
    while flag:
     input_msg = input('input>>>')
     if input_msg == '0':
      break
     s.sendall(input_msg.encode())
     msg = s.recv(1024)
     print(msg.decode())
     
    s.close()

  

