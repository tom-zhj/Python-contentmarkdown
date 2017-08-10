# python 类变量 在多线程下的共享与释放问题

最近被多线程给坑了下，没意识到类变量在多线程下是共享的，还有一个就是没意识到 内存释放问题，导致越累越大

1.python 类变量 在多线程情况 下的 是共享的

2.python 类变量 在多线程情况 下的 释放是不完全的

3.python 类变量 在多线程情况 下没释放的那部分 内存 是可以重复利用的

    
    
    import threading
     import time
      
     class Test:
      
         cache = {}
          
         @classmethod
         def get_value(self, key):
             value = Test.cache.get(key, [])
             return len(value)
      
         @classmethod
         def store_value(self, key, value):
             if not Test.cache.has_key(key):
                 Test.cache[key] = range(value)
             else:
                 Test.cache[key].extend(range(value)) 
             return len(Test.cache[key])
      
         @classmethod
         def release_value(self, key):
             if Test.cache.has_key(key):
                 Test.cache.pop(key)
             return True
      
         @classmethod
         def print_cache(self):
             print 'print_cache:'
             for key in Test.cache:
                 print 'key: %d, value:%d' % (key, len(Test.cache[key]))
      
     def worker(number, value):
         key = number % 5
         print 'threading: %d, store_value: %d' % (number, Test.store_value(key, value))
         time.sleep(10)
         print 'threading: %d, release_value: %s' % (number, Test.release_value(key))
      
     if __name__ == '__main__':
         thread_num = 10
          
         thread_pool = []
         for i in range(thread_num):
             th = threading.Thread(target=worker,args=[i, 1000000])
             thread_pool.append(th)
             thread_pool[i].start()
      
         for thread in thread_pool:
             threading.Thread.join(thread)
          
         Test.print_cache()
         time.sleep(10)
          
         thread_pool = []
         for i in range(thread_num):
             th = threading.Thread(target=worker,args=[i, 100000])
             thread_pool.append(th)
             thread_pool[i].start()
      
         for thread in thread_pool:
             threading.Thread.join(thread)
          
         Test.print_cache()
         time.sleep(10)

总结

公用的数据，除非是只读的，不然不要当类成员变量，一是会共享，二是不好释放。

