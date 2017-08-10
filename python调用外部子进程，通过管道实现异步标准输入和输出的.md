# python调用外部子进程，通过管道实现异步标准输入和输出的

我们通常会遇到这样的需求：通过C++或其他较底层的语言实现了一个复杂的功能模块，需要搭建一个基于Web的Demo，方法查询数据。由于Python语言的强大和
简洁，其用来搭建Demo非常合适，Flask框架和jinja2模块功能为python提供了方便的web开发能力。同时，python能够很方便的同其他语言的代
码交互。因此我们选择python作为开发Demo的工具。假设我们需要调用的模块（提供底层服务）通过标准输入循环读入数据，处理完毕后把结果写出到标出输出，这样
的场景在Linux环境下很常见，依赖于Linux强大的重定向能力。然而，非常不幸的是，底层模块有一个很重的初始化过程，因此我们不能够每次查询请求都去重新生成
调用底层模块的子进程。解决方案就是只生成一次子进程，然后对每个请求通过管道（pipe）来和子进程交互。

Python的subprocess模块可以很容易地生成子进程，类似Linux系统调用fork和exec。subprocess模块的Popen对象可能以非阻塞
的方式调用外部可执行程序，因此我们使用Poen对象来实现需求。如果我们想要把数据写入子进程的标准输入stdin，那么在创建Popen对象的时候就需要指定参数
stdin为subprocess.PIPE；同样，如果我们需要从子进程的标准输出中读取数据，那么在创建Popen对象的时候就需要指定参数stdout为sub
process.PIPE。先看一个简单的例子：

    
    
    from subprocess import Popen, PIPE
    p = Popen('less', stdin=PIPE, stdout=PIPE)
    p.communicate('Line number %d.\n' % x)

communicate函数返回一个二元组(stdoutdata, stderrdata)，包含了子进程的标准输出和标出错误的输出数据。然而，由于Popen对
象的communicate函数会阻塞父进程，同时还会关闭管道，因此每个Popen对象只能调用一次communicate函数，如果有多个请求必须重新生成Pop
en对象（重新初始化子进程），不能满足我们的需求。

因此，我们只有往Popen对象的stdin和stdout对象里写入和读取数据才能实现我们的需求。然而，不幸的是subprocess模块默认情况下只运行在子进
程结束的时候读取一次标准输出。Both subprocess and os.popen* only allow input and output one
time, and the output to be read only when the process terminates.

进过一番研究之后我发现通过fcntl模块的fcntl函数可以把子进程的标准输出改为非阻塞的方式，从而达到我们的目的。这样困扰我许久的问题终于得到了完美解决。
代码如下：

    
    
    #!/usr/bin/python                                                                                                                                                      
    # -*- coding: utf-8 -*-
    # author: weisu.yxd@taobao.com
    from subprocess import Popen, PIPE
    import fcntl, os
    import time
    class Server(object):
      def __init__(self, args, server_env = None):
        if server_env:
          self.process = Popen(args, stdin=PIPE, stdout=PIPE, stderr=PIPE, env=server_env)
        else:
          self.process = Popen(args, stdin=PIPE, stdout=PIPE, stderr=PIPE)
        flags = fcntl.fcntl(self.process.stdout, fcntl.F_GETFL)
        fcntl.fcntl(self.process.stdout, fcntl.F_SETFL, flags | os.O_NONBLOCK)
      def send(self, data, tail = '\n'):
        self.process.stdin.write(data + tail)
        self.process.stdin.flush()
      def recv(self, t=.1, e=1, tr=5, stderr=0):
        time.sleep(t)
        if tr < 1:
            tr = 1 
        x = time.time()+t
        r = ''
        pr = self.process.stdout
        if stderr:
          pr = self.process.stdout
        while time.time() < x or r:
            r = pr.read()
            if r is None:
                if e:
                    raise Exception(message)
                else:
                    break
            elif r:
                return r.rstrip()
            else:
                time.sleep(max((x-time.time())/tr, 0))
        return r.rstrip()
    if __name__ == "__main__":
      ServerArgs = ['/home/weisu.yxd/QP/trunk/bin/normalizer', '/home/weisu.yxd/QP/trunk/conf/stopfile.txt']
      server = Server(ServerArgs)
      test_data = '在云端', '云梯', '摩萨德', 'Alisa', 'iDB', '阿里大数据'
      for x in test_data:
        server.send(x)
        print x, server.recv()

 另外，调用一些外部程序时，可能需要指定相应的环境变量，方式如下：

    
    
      my_env = os.environ
      my_env["LD_LIBRARY_PATH"] = "/path/to/lib"
      server = server.Server(cmd, my_env)

  

