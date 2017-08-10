# 如何让python变得更快？

Python和其他脚本语言通常会被摒弃，因为它们相对于一些类似于C语言的编译型的语言来说效率很低。比如下面的斐波纳契数的例子：

C语言中：

    
    
    int fib(int n){
       if (n < 2)
         return n;
       else
         return fib(n - 1) + fib(n - 2);
    }
    int main() {
        fib(40);
        return 0;

  

Python中：

    
    
    def fib(n):
      if n < 2:
         return n
      else:
         return fib(n - 1) + fib(n - 2)
    fib(40)

  

下面是它们各自的执行时间：

    
    
    $ time ./fib
    3.099s
     
    $ time python fib.py
    16.655s

  

和预期的一样，在这个例子中C语言的执行效率要比Python快5倍。

  

在网络抓取的情况下，执行速度并不是很重要因为瓶颈在于I/O -
下载web页面。但是我在其他环境也想使用Python，所以我们来看一下怎么样提高python的执行速度。

  

首先我们来安装一个python模块：psyco，安装非常简单，只需要执行如下命令：

    
    
    sudo apt-get install python-psyco

或者你是在centos的话，执行：

    
    
    sudo yum install python-psyco

然后我们来验证一下：

    
    
    #引入psyco模块，author: www.pythontab.com
    import psyco
    psyco.full()
    def fib(n):
      if n < 2:
         return n
      else:
         return fib(n - 1) + fib(n - 2)
    fib(40)

哈哈，见证奇迹的时刻！！

    
    
    $ time python fib.py
    3.190s

仅用了3秒，使用psyco模块后python的运行速度和C不相上下！

  

现在我几乎大部分python代码都会加上如下代码，享受psyco所带来的速度提升

    
    
    try:
        import psyco
        psyco.full()
    except ImportError:
        pass # psyco not installed so continue as usual

  

