# python多线程编程4: 死锁和可重入锁

死锁

在线程间共享多个资源的时候，如果两个线程分别占有一部分资源并且同时等待对方的资源，就会造成死锁。尽管死锁很少发生，但一旦发生就会造成应用的停止响应。下面看一
个死锁的例子：

    
    
    # encoding: UTF-8
    import threading
    import time
     
    class MyThread(threading.Thread):
        def do1(self):
            global resA, resB
            if mutexA.acquire():
                 msg = self.name+' got resA'
                 print msg
                  
                 if mutexB.acquire(1):
                     msg = self.name+' got resB'
                     print msg
                     mutexB.release()
                 mutexA.release()
        def do2(self):
            global resA, resB
            if mutexB.acquire():
                 msg = self.name+' got resB'
                 print msg
                  
                 if mutexA.acquire(1):
                     msg = self.name+' got resA'
                     print msg
                     mutexA.release()
                 mutexB.release()
      
         
        def run(self):
            self.do1()
            self.do2()
    resA = 0
    resB = 0
     
    mutexA = threading.Lock()
    mutexB = threading.Lock()
     
    def test():
        for i in range(5):
            t = MyThread()
            t.start()
    if __name__ == '__main__':
        test()

  

执行结果：

  

Thread-1 got resA

Thread-1 got resB

Thread-1 got resB

Thread-1 got resA

Thread-2 got resA

Thread-2 got resB

Thread-2 got resB

Thread-2 got resA

Thread-3 got resA

Thread-3 got resB

Thread-3 got resB

Thread-3 got resA

Thread-5 got resA

Thread-5 got resB

Thread-5 got resB

Thread-4 got resA

  

此时进程已经死掉。

  

可重入锁

更简单的死锁情况是一个线程“迭代”请求同一个资源，直接就会造成死锁：

    
    
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
                mutex.acquire()
                mutex.release()
                mutex.release()
    num = 0
    mutex = threading.Lock()
    def test():
        for i in range(5):
            t = MyThread()
            t.start()
    if __name__ == '__main__':
        test()

  

为了支持在同一线程中多次请求同一资源，python提供了“可重入锁”：threading.RLock。RLock内部维护着一个Lock和一个counter变
量，counter记录了acquire的次数，从而使得资源可以被多次require。直到一个线程所有的acquire都被release，其他的线程才能获得资
源。上面的例子如果使用RLock代替Lock，则不会发生死锁：

    
    
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
                mutex.acquire()
                mutex.release()
                mutex.release()
    num = 0
    mutex = threading.RLock()
    def test():
        for i in range(5):
            t = MyThread()
            t.start()
    if __name__ == '__main__':
        test()

执行结果：

  

Thread-1 set num to 1

Thread-3 set num to 2

Thread-2 set num to 3

Thread-5 set num to 4

Thread-4 set num to 5

  

