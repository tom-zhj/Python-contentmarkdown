# python多线程编程3: 使用互斥锁同步线程

问题的提出

上一节的例子中，每个线程互相独立，相互之间没有任何关系。现在假设这样一个例子：有一个全局的计数num，每个线程获取这个全局的计数，根据num进行一些处理，然
后将num加1。很容易写出这样的代码：

    
    
    # encoding: UTF-8
    import threading
    import time
     
    class MyThread(threading.Thread):
        def run(self):
            global num
            time.sleep(1)
            num = num+1
            msg = self.name+' set num to '+str(num)
            print msg
    num = 0
    def test():
        for i in range(5):
            t = MyThread()
            t.start()
    if __name__ == '__main__':
        test()

  

但是运行结果是不正确的：

  

Thread-5 set num to 2

Thread-3 set num to 3

Thread-2 set num to 5

Thread-1 set num to 5

Thread-4 set num to 4

  

问题产生的原因就是没有控制多个线程对同一资源的访问，对数据造成破坏，使得线程运行的结果不可预期。这种现象称为“线程不安全”。

互斥锁同步

上面的例子引出了多线程编程的最常见问题：数据共享。当多个线程都修改某一个共享数据的时候，需要进行同步控制。

线程同步能够保证多个线程安全访问竞争资源，最简单的同步机制是引入互斥锁。互斥锁为资源引入一个状态：锁定/非锁定。某个线程要更改共享数据时，先将其锁定，此时资
源的状态为“锁定”，其他线程不能更改；直到该线程释放资源，将资源的状态变成“非锁定”，其他的线程才能再次锁定该资源。互斥锁保证了每次只有一个线程进行写入操作
，从而保证了多线程情况下数据的正确性。

threading模块中定义了Lock类，可以方便的处理锁定：

#创建锁

mutex = threading.Lock()

#锁定

mutex.acquire([timeout])

#释放

mutex.release()

其中，锁定方法acquire可以有一个超时时间的可选参数timeout。如果设定了timeout，则在超时后通过返回值可以判断是否得到了锁，从而可以进行一些
其他的处理。

使用互斥锁实现上面的例子的代码如下：

    
    
    import threading
    import time
     
    class MyThread(threading.Thread):
        def run(self):
            global num 
            time.sleep(1)
     
            if mutex.acquire(1):  
                num = num+1
                msg = self.name+' set num to '+str(num)
                print msg
                mutex.release()
    num = 0
    mutex = threading.Lock()
    def test():
        for i in range(5):
            t = MyThread()
            t.start()
    if __name__ == '__main__':
        test()

  

运行结果：

  

Thread-3 set num to 1

Thread-4 set num to 2

Thread-5 set num to 3

Thread-2 set num to 4

Thread-1 set num to 5

可以看到，加入互斥锁后，运行结果与预期相符。

同步阻塞

当一个线程调用锁的acquire()方法获得锁时，锁就进入“locked”状态。每次只有一个线程可以获得锁。如果此时另一个线程试图获得这个锁，该线程就会变为
“blocked”状态，称为“同步阻塞”（参见多线程的基本概念）。

直到拥有锁的线程调用锁的release()方法释放锁之后，锁进入“unlocked”状态。线程调度程序从处于同步阻塞状态的线程中选择一个来获得锁，并使得该线
程进入运行（running）状态。

互斥锁最基本的内容就是这些，下一节将讨论可重入锁（RLock)和死锁问题。

  

