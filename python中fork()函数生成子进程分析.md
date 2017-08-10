# python中fork()函数生成子进程分析

   python的os
module中有fork()函数用于生成子进程，生成的子进程是父进程的镜像，但是它们有各自的地址空间，子进程复制一份父进程内存给自己，两个进程之
间的执行是相互独立的，其执行顺序可以是不确定的、随机的、不可预测的，这点与多线程的执行顺序相似。

    
    
    import os
    def child():
        print 'A new child:', os.getpid()
        print 'Parent id is:', os.getppid()
        os._exit(0)
    def parent():
        while True:
            newpid=os.fork()
            print newpid
            if newpid==0:
                child()
            else:
                pids=(os.getpid(),newpid)
                print "parent:%d,child:%d"%pids
                print "parent parent:",os.getppid()       
            if raw_input()=='q':
                break
    parent()

    在我们加载了os模块之后，我们parent函数中fork()函数生成了一个子进程，返回值newpid有两个，一个为0，用以表示子进程，一个是大于
0的整数，用以表示父进程，这个常数正是子进程的pid.
通过print语句我们可以清晰看到两个返回值。如果fork()返回值是一个负值，则表明子进程生成不成功(这个简单程序中没有考虑这种情况)。如果
newpid==0,则表明我们进入到了子进程，也就是child()函数中，在子进程中我们输出了自己的id和父进程的id。如果进入了else语句， 则表明ne
wpid>0，我们进入到父进程中，在父进程中os.getpid()得到自己的id,fork()返回值newpid表示了子进程的id,同时我们输出了父进程的父
进程的id. 通过实验我们可以看到if和else语句的执行顺序是不确定的，子、父进程的执行顺序由操作[系统](http://www.2cto.com/os/
)的调度算法来决定。  

  

