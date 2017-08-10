# 常用的 Python 调试工具，Python开发必读

以下是我做调试或分析时用过的工具的一个概览。如果你知道有更好的工具，请在评论中留言，可以不用很完整的介绍。

###### 日志

没错，就是日志。再多强调在你的应用里保留足量的日志的重要性也不为过。你应当对重要的内容打日志。如果你的日志打的足够好的话，单看日志你就能发现问题所在。那样可
以节省你大量的时间。

如果一直以来你都在代码里乱用 print 语句，马上停下来。换用logging.debug。以后你还可以继续复用，或是全部停用等等。

###### 跟踪

有时更好的办法是看执行了哪些语句。你可以使用一些IDE的调试器的单步执行，但你需要明确知道你在找那些语句，否则整个过程会进行地非常缓慢。

标准库里面的trace模块，可以打印运行时包含在其中的模块里所有执行到的语句。(就像制作一份项目报告)

    
    
    
    python -mtrace –trace script.py

这会产生大量输出（执行到的每一行都会被打印出来，你可能想要用grep过滤那些你感兴趣的模块）.

比如：

    
    
    
    python -mtrace –trace script.py | egrep '^(mod1.py|mod2.py)'

###### 调试器

以下是如今应该人尽皆知的一个基础介绍：

    
    
    
    import pdb
    pdb.set_trace() # 开启pdb提示

或者

    
    
    
    try:
    （一段抛出异常的代码）
    except:
        import pdb
        pdb.pm() # 或者 pdb.post_mortem()
    　　或者(输入 c 开始执行脚本)
    　　

python -mpdb script.py

在输入-计算-输出循环(注：REPL，READ-EVAL-PRINT-LOOP的缩写)环境下，可以有如下操作：

c or continue

q or quit

l or list, 显示当前步帧的源码

w or where,回溯调用过程

d or down, 后退一步帧（注：相当于回滚）

u or up, 前进一步帧

(回车), 重复上一条指令

其余的几乎全部指令（还有很少的其他一些命令除外）,在当前步帧上当作python代码进行解析。

如果你觉得挑战性还不够的话，可以试下smiley，-它可以给你展示那些变量而且你能使用它来远程追踪程序。

###### 更好的调试器

pdb的直接替代者：

ipdb(easy_install ipdb) – 类似ipython(有自动完成，显示颜色等)

pudb(easy_install pudb) – 基于curses（类似图形界面接口），特别适合浏览源代码

###### 远程调试器

安装方式:

sudo apt-get install winpdb

用下面的方式取代以前的pdb.set_trace()：

    
    
    
    import rpdb2
    rpdb2.start_embedded_debugger("secretpassword")

现在运行winpdb,文件-关联

不喜欢Winpdb?也可以直接包装PDB在TCP之上运行！

这样做：

    
    
    
    import loggging
    class Rdb(pdb.Pdb):
        """
        This will run pdb as a ephemeral telnet service. Once you connect no one
        else can connect. On construction this object will block execution till a
        client has connected.
     
        Based on https://github.com/tamentis/rpdb I think ...
     
        To use this::
     
            Rdb(4444).set_trace()
     
        Then run: telnet 127.0.0.1 4444
        """
        def __init__(self, port=0):
            self.old_stdout = sys.stdout
            self.old_stdin = sys.stdin
            self.listen_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            self.listen_socket.bind(('0.0.0.0', port))
            if not port:
                logging.critical("PDB remote session open on: %s", self.listen_socket.getsockname())
                print >> sys.__stderr__, "PDB remote session open on:", self.listen_socket.getsockname()
                sys.stderr.flush()
            self.listen_socket.listen(1)
            self.connected_socket, address = self.listen_socket.accept()
            self.handle = self.connected_socket.makefile('rw')
            pdb.Pdb.__init__(self, completekey='tab', stdin=self.handle, stdout=self.handle)
            sys.stdout = sys.stdin = self.handle
     
        def do_continue(self, arg):
            sys.stdout = self.old_stdout
            sys.stdin = self.old_stdin
            self.handle.close()
            self.connected_socket.close()
            self.listen_socket.close()
            self.set_continue()
            return 1
     
        do_c = do_cont = do_continue
     
    def set_trace():
        """
        Opens a remote PDB on first available port.
        """
        rdb = Rdb()
        rdb.set_trace()

只想要一个REPL环境？试试IPython如何？

如果你不需要一个完整齐全的调试器，那就只需要用一下的方式启动一个IPython即可：

    
    
    
    import IPython
    IPython.embed()

###### 标准linux工具

我常常惊讶于它们竟然远未被充分利用。你能用这些工具解决很大范围内的问题：从性能问题（太多的系统调用，内存分配等等）到死锁，网络问题，磁盘问题等等。

其中最有用的是最直接的strace，只需要运行 sudo strace -p 12345 或者 strace -f 指令(-f
即同时追踪fork出来的子进程)，这就行了。输出一般会非常大，所以你可能想要把它重定向到一个文件以便作更多的分析（只需要加上 &> 文件名）。

再就是ltrace,有点类似strace，不同的是，它输出的是库函数调用。参数大体相同。

还有lsof 用来指出你在ltrace/strace中看到的句柄数值的意义。比如：

lsof -p 12345

更好的跟踪

使用简单而可以做很多事情-人人都该装上htop！

sudo apt-get install htop

sudo htop

现在找到那些你想要的进程，再输入：

s - 代表系统调用过程(类似strace)

L - 代表库调用过程(类似ltrace)

l - 代表lsof

　　监控

没有好的持续的服务器监控，但是如果你曾遇到一些很诡异的情况，诸如为什么一切都运行的那么慢，那些系统资源都干什么去了，。。。等这些问题，想弄明白却又
无处下手之际，不必动用iotop,iftop,htop,iostat,vmstat这些工具，就用dstat吧！它可以做之前我们提过的大部分工作可
以做的事情，而且也许可以做的更好！

它会用一种紧凑的，代码高亮的方式（不同于iostat,vmstat）向你持续展示数据，你还经常可以看到过去的数据（不同于iftop,iostop,htop）
。

只需运行：

dstat --cpu --io --mem --net --load --fs --vm --disk-util --disk-tps
--freespace --swap --top-io --top-bio-adv

很可能有一种更简短的方式来写上面这条命令，

这是一个相当复杂而又强大的工具，但是这里我只提到了一些基本的内容（安装以及基础的命令）

sudo apt-get install gdb python-dbg

zcat /usr/share/doc/python2.7/gdbinit.gz > ~/.gdbinit

用python2.7-dbg 运行程序：

sudo gdb -p 12345

现在使用：

bt - 堆栈跟踪（C 级别）

pystack - python 堆栈跟踪,不幸的是你需要有~/.gdbinit 并且使用python-dbg

c - 继续

　　发生段错误？用faulthandler ！

  

　　python 3.3版本以后新增的一个很棒的功能，可以向后移植到python2.x版本。只需要运行下面的语句，你就可以大抵知道什么原因引起来段错误。

import faulthandler

faulthandler.enable()

###### 内存泄露

嗯，这种情况下有很多的工具可以使用，其中有一些专门针对WSGI的程序比如Dozer，但是我最喜欢的当然是objgraph。使用简单方便，让人惊讶！

它没有集成WSGI或者其他，所以你需要自己去发现运行代码的方法，像下面这样：

import objgraph

objs = objgraph.by_type("Request")[:15]

objgraph.show_backrefs(objs, max_depth=20, highlight=lambda v: v in objs,

  

filename="/tmp/graph.png")

Graph written to /tmp/objgraph-zbdM4z.dot (107 nodes)

Image generated as /tmp/graph.png

你会得到像这样一张图（注意：它非常大)。你也可以得到一张点输出。

###### 内存使用

有时你想少用些内存。更少的内存分配常常可以使程序执行的更快，更好，用户希望内存合适好用)

有许多可用的工具，但在我看来最好用的是pytracemalloc。与其他工具相比，它开销非常小（不需要依赖于严重影响速度的sys.settrace）而且输出
非常详尽。但安装起来比较痛苦，你需要重新编译python，但有了apt，做起来也非常容易。

只需要运行这些命令然后去吃顿午餐或者干点别的：

apt-get source python2.7

cd python2.7-*

wget? https://github.com/wyplay/pytracemalloc/raw/master/python2.7_track_free_
list.patch

patch -p1 < python2.7_track_free_list.patch

debuild -us -uc

cd ..

sudo dpkg -i python2.7-minimal_2.7*.deb python2.7-dev_*.deb

接着安装pytracemalloc （注意如果你在一个virtualenv虚拟环境下操作，你需要在重新安装python后再次重建 – 只需要运行
virtualenv myenv）

pip install pytracemalloc

现在像下面这样在代码里包装你的应用程序

    
    
    
    import tracemalloc, time
    tracemalloc.enable()
    top = tracemalloc.DisplayTop(
        5000, # log the top 5000 locations
        file=open('/tmp/memory-profile-%s' % time.time(), "w")
    )
    top.show_lineno = True
    try:
        # code that needs to be traced
    finally:
        top.display()

　　输出会像这样：

  

2013-05-31 18:05:07: Top 5000 allocations per file and line

 #1: .../site-packages/billiard/_connection.py:198: size=1288 KiB, count=70
(+0),

average=18 KiB

 #2: .../site-packages/billiard/_connection.py:199: size=1288 KiB, count=70
(+0),

average=18 KiB

 #3: .../python2.7/importlib/__init__.py:37: size=459 KiB, count=5958 (+0),

average=78 B

 #4: .../site-packages/amqp/transport.py:232: size=217 KiB, count=6960 (+0),

average=32 B

 #5: .../site-packages/amqp/transport.py:231: size=206 KiB, count=8798 (+0),

average=24 B

 #6: .../site-packages/amqp/serialization.py:210: size=199 KiB, count=822
(+0),

average=248 B

 #7: .../lib/python2.7/socket.py:224: size=179 KiB, count=5947 (+0),
average=30

B

 #8: .../celery/utils/term.py:89: size=172 KiB, count=1953 (+0), average=90 B

 #9: .../site-packages/kombu/connection.py:281: size=153 KiB, count=2400 (+0),

average=65 B

 #10: .../site-packages/amqp/serialization.py:462: size=147 KiB, count=4704

(+0), average=32 B

　　…

  

