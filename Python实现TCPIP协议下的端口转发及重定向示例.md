# Python实现TCP/IP协议下的端口转发及重定向示例

首先，我们用webpy写一个简单的网站，监听8080端口，返回“Hello, EverET.org”的页面。

然后我们使用我们的forwarding.py，在80端口和8080端口中间建立两条通信管道用于双向通信。

此时，我们通过80端口访问我们的服务器。

浏览器得到：

![](http://www.pythontab.com/uploadfile/2016/0615/20160615075451348.png)

然后，我们在forwarding.py的输出结果中可以看到浏览器和webpy之间的通信内容。

![](http://www.pythontab.com/uploadfile/2016/0615/20160615075452819.png)

  

  

代码：

    
    
    #!/usr/bin/env python
    import sys, socket, time, threading
    loglock = threading.Lock()
    def log(msg):
      loglock.acquire()
      try:
        print '[%s]: \n%s\n' % (time.ctime(), msg.strip())
        sys.stdout.flush()
      finally:
        loglock.release()
    class PipeThread(threading.Thread):
      def __init__(self, source, target):
        threading.Thread.__init__(self)
        self.source = source
        self.target = target
      def run(self):
        while True:
          try:
            data = self.source.recv(1024)
            log(data)
            if not data: break
            self.target.send(data)
          except:
            break
        log('PipeThread done')
    class Forwarding(threading.Thread):
      def __init__(self, port, targethost, targetport):
        threading.Thread.__init__(self)
        self.targethost = targethost
        self.targetport = targetport
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.sock.bind(('0.0.0.0', port))
        self.sock.listen(10)
      def run(self):
        while True:
          client_fd, client_addr = self.sock.accept()
          target_fd = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
          target_fd.connect((self.targethost, self.targetport))
          log('new connect')
          # two direct pipe
          PipeThread(target_fd, client_fd).start()
          PipeThread(client_fd, target_fd).start()
    if __name__ == '__main__':
      print 'Starting'
      import sys
      try:
        port = int(sys.argv[1])
        targethost = sys.argv[2]
        try: targetport = int(sys.argv[3])
        except IndexError: targetport = port
      except (ValueError, IndexError):
        print 'Usage: %s port targethost [targetport]' % sys.argv[0]
        sys.exit(1)
      #sys.stdout = open('forwaring.log', 'w')
      Forwarding(port, targethost, targetport).start()

  

