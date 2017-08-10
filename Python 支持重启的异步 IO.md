# Python 支持重启的异步 IO

摘要

  

这是一份从Python3.3开始的Python3异步I/O提议。研究从PEP 3153缺失的具体提议。
这提议包括了一个可插入式的事件循环API，传输和与Twisted相似的协议抽象，以及来自(PEP 380)
基于yield的更高级的调度器。一份作品里的参考实现，它的代码命名为Tulip（Tulip的repo的链接放在文章最后的参考文献分段里）。

介绍

  

事件循环常用于互操作性较高的地方。对于像Twisted、Tornado或者ZeroMQ这类（基于Python3.3的）框架，它应该是容易去根据框架的需求通过
轻量级封装或者代理去适配默认的事件循环实现，或者是用它们自己的事件循环实现去替代默认的实现。（一些像Twisted的框架，拥有多种的事件循环实现。由于这些实
现都具有统一的接口，所以这应该不会成为问题。）

事件循环甚至有可能存在两个不同的第三方框架交互，通过共享默认事件循环实现（各自使用自己的适配器），或者是通过共享其中一个框架的事件循环实现。在后者，两种不同
级别的适配可能会存在（从框架A的事件循环到标准事件循环接口，然后从标准的再到框架B的事件循环）。被使用的事件循环实现应该在主程序的控制之下（尽管提供了事件循
环可以选择的默认策略）。

因此，两个单独的API被定义：

获取和设置当前事件的循环对象；

一个确认事件循环对象的接口和它的最低保证

一个事件循环实现可能提供额外的方法和保证。

事件循环接口不取决于产量，相反，它使用一个回调，额外接口（传输协议和协议）以及期货（？）的组合。后者类似于在PEP3148中定义的接口，但有不同的实现而且不
绑定到线程。特别是，它们没有wait()方法，用户将使用回调。

对于那些不喜欢用回调函数的人（包括我），（Python）提供了一个写异步I/O代码的调度器作为协同程序，它使用了PEP 380的yield
from表达式。这个调度器并不是可插入的；可插入性存在于事件循环级别，同时调度器也应该工作在任何符合标准的事件循环实现上。

对于那些在使用协同程序和其他异步框架的代码的互操作性，调度器有一个行为上类似Future的Task类。一个在事件循环级别进行交互操作的框架能够通过加入一个回
调函数到Future中，同时等待一个Future来完成。同样的，调度器提供一个操作来挂起协同程序，直到回调函数被调用。

通过事件循环接口来为线程间交互操作提供限制；（Python里）有一个API能够提交一个函数到一个能够返回兼容事件循环的Future的执行器（看 PEP
3148）。

没有目的的

  

像Stackless Python或 greenlets/gevent 的系统互操作性不是本教程的目的。

规范

  

依赖

  

Python3.3是必需的。不需超过Python3.3范围的新语言或标准库。不需要第三方模块或包。

模块命名空间

  

这里的规范会放在一个新的顶层包。不同的组件会放在这个包的不同子模块里。包会从各自的子模块中导入常用的API，同时使他们能作为包的可用属性（类似电子邮件包的做
法）。

顶层包的名字目前还没指定。参考实现使用“tulip”来命名，但是这个名字可能会在这个实现加入到标准库的时候改变为其他更为烦人的名字（有希望在Python3.
4中）。

在烦人的名字选好之前，这篇教程会使用“tulip”作为顶层包的名字。假定没有给定模块名的类和函数都通过顶层包来访问。

事件循环策略：获取和设置事件循环

  

要获取当前的事件循环，可以使用get_event_loop()。这函数返回一个在下面有定义的EventLoop类的实例或者一个等价的对象。get_event
_loop()可能根据当前线程返回不同的对象，或者根据其他上下文的概念返回不同对象。

要设置当前事件循环，可以使用set_event_loop(event_loop)，这里的event_loop是EventLoop类的实例或者等价的实例对象。
这里使用与get_event_loop()相同的上下文的概念。

还有第三个策略函数：new_event_loop()，有利于单元测试和其他特别的情况，它会创建和返回一个新的基于该策略的默认规则的EventLoop实例。要
使它成为当前的事件循环，你需要调用set_event_loop()。

要改变上述三个函数的工作方式（包括他们的上下文的概念），可以通过调用set_event_loop_policy(policy)，其中参数policy是一个事
件循环策略对象。这个策略对象可以是任何包含了类似上面描述的函数表现（get_event_loop(),set_event_loop(event_loop)和
new_event_loop()）的对象。默认的事件循环策略是DefaultEventLoopPolicy类的一个实例。当前事件循环策略对象能够通过调用ge
t_event_loop_policy()来取回。

一个事件循环策略没强制要求只能有一个事件循环存在。默认的事件循环策略也没强制要求这样做，但是它强制要求每个线程只能有一个事件循环。

事件循环接口

  

关于时间：在 Python 中，所有的超时（timeout），间隔（interval）和延时（delay）都是以秒来计算的，可以是整型也可以是浮点型。时钟的
精准度依赖于具体的实现；默认使用 time.monotonic()。

关于回调（callbacks）和处理函数（handlers）：如果一个函数接受一个回调函数和任意个数的变量作为参数，那么你也可以用一个处理函数对象（Hand
ler）来替换回调函数。这样的话就不需要再传递那些参数。这个处理函数对象应该是一个立即返回的函数（fromcall_soon()），而不是延迟返回的（fro
mcall_later()）。如果处理函数已经取消，那么这个调用将不起作用。

一个符合标准的事件循环对象拥有以下的方法：

run()。 执行事件循环，知道没啥好做了。具体的意思是：

除了取消调用外，没有更多通过call_later(),call_repeatedly(),call_soon(),
orcall_soon_threadsafe()这些方法调度的调用。

没有更多的注册了的文件描述符。 当它关闭的时候会由注册方来注销文件描述符。

备注:直到遇到终止条件或者调用stop()，run()会一直阻塞。

备注: 如果你使用call_repeatedly()来执行一个调用，run()不会在你调用stop()前退出。

需要详细说明： 有多少类似的真正需要我们做的？

run_forever()。直到调用stop()前一直运行事件循环。

run_until_complete(future,
timeout=None)。在Future完成前一直运行事件循环。如果给出了timeout的值，它会等待timeout的时间。
如果Future完成了，它的结果会返回 或者它的异常抛出；如果在超时前完成Future， 或者stop()被调用,会抛出TimeoutError
（但Future不会被取消）. 在事件循环已经在运行的时候，这个方法不能调用。

备注: 这个API更多用来做测试或者类似的工作。 它不应该用作从yield from 表达式的future替代品或其他等待一个Future的方法。 (例如
注册一个完成的回调)。

run_once(timeout=None)。运行事件循环一段事件。 如果给出了timeout的值， I/O轮询会阻塞一段时间； 否则,
I/O轮询不会受时间约束。

备注：准确来说，这里做了多少工作是根据具体实现的。
一个约束是：如果一个使用call_soon()来直接调度自己，会导致死顺坏，run_once()仍然会返回。

stop()。尽可能快地停止事件循环。随后可以使用run()重启循环（或者的一个变体）。

备注:
有多块来停止是根据它具体实现。所有在stop()前已经在运行的直接回调函数必定仍在运行，但是在stop()调用后的调度的回调函数（或者延迟运行的）不会运行。

close()。关闭事件循环，释放它所保持的所有资源，例如被epoll()或kqueue()使用的文件描述符。这个方法不应该在事件循环运行期间调用。它可以被
多次调用。

call_later(delay, callback, *args)。为callback(*args)安排延迟大约delay秒后调用，一旦，除非被取消了。返
回一个Handler对象代表回调函数，Handler对象的cancel()方法常用来取消回调函数。

call_repeatedly(interval, callback, **args)。和call_later()类似，但是会在每个interval秒中重复
调用回调函数，直到返回的Handler被取消。第一次调用是在interval秒内。

call_soon(callback, *args)。类似call_later(0, callback, *args)。

call_soon_threadsafe(callback, *args)。类似call_soon(callback, *args)，但是当事件循环在阻塞等
待IO的时候在另外的线程调用，事件循环的阻塞会被取消。这是唯一安全地从另外的线程调用的方法。（要在一个线程安全的方式去延迟一段时间调度回调函数，你可以使用e
v.call_soon_threadsafe(ev.call_later,when,callback,*args)。）但它在信号处理器中调用并不安全（因为它
可以使用锁）。

add_signal_handler(sig, callback, *args)。无论什么时候接收到信号 ``sigis ， callback(*args)
会被安排调用。返回一个能来取消信号回调函数的Handler。（取消返回的处理器回导致在下个信号到来的时候调用remove_signal_handler()。
优先明确地调用remove_signal_handler()。）为相同的信号定义另外一个回到函数来替代之前的handler（每个信号只能激活一个handle
r）。sig参数必需是一个在信号模块里定义的有效的信号值。如果信号不能处理，这会抛出一个异常：如果它不是一个有效的信号或者如果它是一个不能捕获的信号（例如S
IGKILL），会抛出ValueError。如果这个特别的事件循环实例不能处理信号（因为信号是每个处理器的全局变量，只有在主线程的事件循环才能处理这些信号）
，它会抛出RuntimeError。

remove_signal_handler(sig)。为信号sig移除handler，当有设置的时候。抛出和add_signal_handler()一样的异
常（除了在不能不错的信号时返回False代替抛出RuntimeError）。如果handler移除成功，返回True，如果没有设置handler则返回Fal
se。

一些符合标准接口返回Future的方法：

wrap_future(future)。 这里需要在PEP 3148 描述的Future
（例如一个concurrent.futures.Future的实例） 同时返回一个兼容事件循环的Future （例如, 一个tulip.Future实例）。

run_in_executor(executor, callback, *args)。安排在一个执行器中调用callback(*args) （请看 PEP
3148）。返回的Future的成功的结果是调用的返回值。 这个方法等价于wrap_future(executor.submit(callback,
*args))。 如果没有执行器,则会是也能够一个默认为5个线程的ThreadPoolExecutor。

set_default_executor(executor). 设置一个被run_in_executor()使用的默认执行器。

getaddrinfo(host, port, family=0, type=0, proto=0, flags=0). 类似socket.getaddri
nfo()函数，但是返回一个Future。Future的成功结果为一列与socket.getaddrinfo()的返回值有相同格式数据。 默认的实现通过ru
n_in_executor()来调用socket.getaddrinfo()，但其他实现可能会选择使用他们自己的DNS查找。可选参数必需是指定的关键字参数。

getnameinfo(sockaddr, flags=0). 类似socket.getnameinfo()，但返回一个Future。
Future的成功的结果将会是一个(host, port)的数组。 与forgetaddrinfo()有相同的实现。

create_connection(protocol_factory, host, port, **kwargs).
使用给定的主机和端口创建一个流链接。这会创建一个依赖Transport的实现来表示链接，
然后调用protocol_factory()来实例化（或者取回）用户的Protocol实现，
然后把两者绑定到一起。（看下面对Transport和Protocol的定义。） 用户的Protocol实现通过调用无参数(*)的protocol_facto
ry()来创建或者取回。返回值是Future，它的成功结果是(transport, protocol)对； 如果有错误阻止创建一个成功的链接，Future会
包含一个适合的异常集。注意，当Future完成的适合，协议的connection_made()方法不会调用；那会发生在链接握手完成的适合。

(*) 没有要求protocol_factory是一个类。如果你的协议类需要定义参数传递到构造函数，你可以使用lambda或者functool.partia
l()。你也可以传入一个之前构造好的Protocol实例的lambda。

可选关键参数:

family,proto,flags：地址簇，协议，
和混合了标志的参数传递到getaddrinfo()。这些全是默认为0的。（(socket类型总是SOCK_STREAM。）

ssl：传入True来创建一个SSL传输（通过默认一个无格式的TCP来建立）。或者传入一个ssl.SSLConteext对象来重载默认的SSL上下文对象来使
用。

start_serving(protocol_factory, host, port, **kwds)。进入一个接收连接的循环。返回一个Future，一旦循
环设定到服务即完成；它的返回值为是None。每当接收一个链接，无参数(*)的protocol_factory被调用来创建一个Protocol，一个代表了链接
网络端的Transport会被创建，以及两个对象通过调用protocol.connection_made(transport)绑定到一起。

（*）看上面对create_connection()的补充说明。 然而，
因为protocol_factory()只会在每个新进来的连接调用一次，所以推荐在它每次调用的时候返回一个新的Protocol对象。

可选关键参数:

family,proto,flags：地址簇，协议，
和混合了标志的参数传递到getaddrinfo()。这些全是默认为0的。（(socket类型总是SOCK_STREAM。）

补充: 支持SSL吗? 我并不知道怎样来异步地支持（SSL），我建议这需要一个证书。

补充:也许可以使一个Future的结果对象能够用来控制服务循环，例如停止服务，终止所有的活动连接，以及（如果支持）调整积压（的服务）或者其他参数？它也可以有
一个API来查询活动连接。另外，如果循环由于错误而停止服务，或者如果不能启动，则返回一个仅仅完成了的Future（子类？）？取消了它可能会导致停止循环。

补充：一些平台可能没兴趣实现所有的这些方法， 例如 移动APP就对start_serving()不太感兴趣。
（尽管我的iPad上有一个Minecraft的服务器...）

以下这些注册文件描述符的回调函数的方法不是必需的。如果没有实现这些方法，访问这些方法（而不是调用它们）时会返回属性错误（AttributeError）。默认
的实现提供了这些方法，但是用户一般不会直接用到它们，只有传输层独家使用。同样，在 Windows 平台，这些方法不一定实现，要看到底是否使用了 select
或 IOCP 的事件循环模型。这两个模型接收整型的文件描述符，而不是 fileno() 方法返回的对象。文件描述符最好是可查询的，例如，磁盘文件就不行。

add_reader(fd, callback, *args). 在文件描述符 fd 准备好可以进行读操作时调用指定的回调函数
callback(*args)。返回一个处理函数对象，可以用来取消回调函数。注意，不同于
call_later()，这个回调函数可以被多次调用。在同一个文件描述符上再次调用 add_reader()
将会取消之前设置的回调函数。注意：取消处理函数有可能会等到处理函数调用后。如果你要关闭 fd，你应该调用
remove_reader(fd)。（TODO：如果已经设置了处理函数，抛出一个异常）。

add_writer(fd, callback, *args).  类似 add_reader()，不过是在可以写操作之前调用回调函数。

remove_reader(fd). 为文件描述符 fd
删除已经设置的读操作回调函数。如果没有设置回调函数，则不进行操作。（提供这样的替代接口是因为记录文件描述符比记录处理函数更方便简单）。删除成功则返回
True，失败则返回 False。

remove_writer(fd).  为文件描述符 fd 删除已经设置的写操作回调函数。

未完成的：如果一个文件描述符里面包含了多个回调函数，那应该怎么办呢？目前的机制是替换之前的回调函数，如果已经注册了回调函数则应该招聘异常。

接下来下面的方法在socket的异步I/O中是可选的。他们是替代上面提到的可选方法的，目的是在Windows的传输实现中使用IOCP（如果事件循环支持）。s
ocket参数必需是唔阻塞socket。

sock_recv(sock, n)。从套接字sock中接收字节。返回一个Future，Future在成功的时候会是一个字节对象。

sock_sendall(sock, data)。发送字节数据到套接字sock。返回一个Future，Future的结果在成功后会是None。（补充：让它去
模拟sendall()或send()会更好吗？但是我认为sendall()——也许它扔应该命名为send()?)

sock_connect(sock, address)。连接到给定的地址。返回一个Future，Future的成功结果是None。

sock_accept(sock)。从socket中接收一个链接。这socket必需在监听模式以及绑定到一个定制。返回一个Future，Future的成功结
果会是一个（conn，peer）的数组，conn是一个已连接的无阻塞socket以及peer是对等的地址。（补充：人们告诉我这个API风格对于高水平的服务器
是很慢的。所以上面也有start_sering()。我们还需要这个吗？）

补充：可选方法都不是太好的。也许这些都是需要的？它仍然依赖于平台的更有效设置。另外的可能是：文档标注这些“仅提供给传输”，然后其他的“可提供给任何的情况”。

回调顺序

  

当在同一时间调度两个回调函数时，它们会按照注册的顺序去执行。例如：

ev.call_soon(foo)

ev.call_soon(bar)

保证foo()会在bar()执行。

如果使用call_soon()，即使系统时钟要逆行，这个保证还是成立的。这同样对call_later(0,callback,*args)有效。然而，如果在系
统时钟要逆行下，零延迟地使用call_later()，那就无法得到保证了。（一个好的事件循环实现应该使用time.monotonic()来避免系统时钟逆行的
情况导致的问题。参考 PEP 418 。）

上下文

  

所有的事件循环都有上下文的概念。对于默认的事件循环实现来说，上下文就是一个线程。一个事件循环实现应该在相同的上下问中运行所有的回调。一个事件循环实现应该在同
一时刻只运行一个回调，所以，回调要负责保持与相同事件循环里调度的其他回调自动互斥。

异常

  

在Python里有两类异常：从Exception类到处的和从BaseException导出的。从Exception导出的异常通常能适当地被捕获和处理；例如，
异常会通过Future传递，以及当他们在一个回调了出现时，会被记录和忽略。

然而，从BaseException到处的异常从来不会被捕获到，他们通常带有一个错误回溯信息，同时导致程序终止。（这类的例子包括KeyboardInterru
pt和SystemExit；如果把这些异常与其他大部分异常同样对待，那时不明智的）。

Handler类

  

有各样注册回调函数的方法（例如call_later())都会返回一个对象来表示注册，改对象能够用来取消回调函数。尽管用户从来不用去实例化这个类，但还是想要给
这个对象一个好的名字：Handler。这个类有一个公用的方法：

cancel(). 尝试取消回调函数。 补充：准确的规范。

只读的公共属性：

callback。 要被调用的回调函数。

args。调用回调函数的参数数组。

cancelled。如果cancel()表调用了，它的值为True。

  

要注意的是一些回调函数（例如通过call_later()注册的）意味着只会被调用一次。其他的（如通过add_reader()注册的）意味着可以多次被调用。

补充：一个调用回调函数的API（是否有必要封装异常处理）？它是不是要记录自己被调用了多少次？也许这个API应该是_call_()这样？（但它应该抑制异常。）

补充：当回调函数在调度的时候有没有一些公共的属性来记录那些实时的值？（因为这需要一些方法来把它保存到堆里面的。）

Futures

  

ulip.Future 特意设计成和 PEP 3148中的 concurrent.futures.Future 类似，只是有细微的不同。这个PEP中谈及
Future 时，都是指 tulip.Future，除非明确指定是
concurrent.futures.Future。tulip.Future支持的公开API如下，同时也指出了和 PEP 3148 的不同：

cancel().  如果该 Future 已经完成（或者被取消了），则返回 False。否则，将该 Future
的状态更改成取消状态（也可以理解成已完成），调度回调函数，返回 True。

cancelled(). 如果该 Future 已经被取消了，返回 True。

running(). 总是返回False。和 PEP 3148 不同，这里没有 running 状态。

done(). 如果该Future已经完成了，返回True。注意：取消了的Future也认为是已经完成了的（这里和其他地方都是这样）。

result(). 返回 set_result() 设置的结果，或者返回 set_exception()
设置的异常。如果已经被取消了，则抛出CancelledError。和 PEP 3148
不同，这里没有超时参数，并不等待。如果该Future尚未完成，则抛出一个异常。

exception(). 同上，返回的是异常。

add_done_callback(fn).  添加一个回调函数，在Future完成（或者被取消）时运行。如果该Future已经完成（或者被取消），通过
call_soon() 来调度回调函数。不同于 PEP
3148，添加的回调函数不会立即被调用，且总是会在调用者的上下文中运行。（典型地，一个上下文是一个线程）。你可以理解为，使用 call_soon()
来调用该回调函数。注意：添加的回调函数（不同于本PEP其他的回调函数，且忽略下面"回调风格（Callback Style）"小节的约定）总会接收到一个
Future 作为参数，且这个回调函数不应该是 Handler 对象。

set_result(result).
T该Future不能处于完成（或者取消）状态。这个方法将使当前Future进入完成状态，并准备调用相关的回调函数。不同于 PEP
3148：这是一个公开的API。

set_exception(exception).  同上，设置的是异常。

内部的方法 set_running_or_notify_cancel() 不再被支持；现在已经没有方法直接设置成 running  状态。

这个PEP定义了以下的异常:

InvalidStateError. 当调用的方法不接受这个 Future 的状态时，将会抛出该异常（例如：在一个已经完成的 Future
中调用set_result() 方法，或者在一个未完成的 Future 中调用 result()方法）。

InvalidTimeoutError. 当调用 result() 或者 exception() 时传递一个非零参数时抛出该异常。

CancelledError.  concurrent.futures.CancelledError 的别名。在一个已经取消的 Future 上面调用
result() 或 exception() 方法时抛出该异常。

TimeoutError. concurrent.futures.TimeoutError 的别名。有可能由
EventLoop.run_until_complete() 方法抛出。

创建一个 Future 时将会与默认的事件循环关联起来。（尚未完成的：允许传递一个事件循环作为参数?）。

concurrent.futures 包里面的 wait() 和 as_completed() 方法不接受 tulip.Future
对象作为参数。然而，有类似的 API tulip.wait() 和 tulip.as_completed()， 如下所述。

在子程序（coroutine）中可以将 tulip.Future 对象应用到 yield from 表达式中。这个是通过 Future 中的
__iter__() 接口实现的。请参考下面的“子程序和调度器”小节。

当 Future 对象被回收时，如果有相关联的异常但是并没有调用 result() 、 exception() 或 __iter__()
方法（或者说产生了异常但是还没有抛出），那么应该将该异常记录到日志中。TBD：记录成什么级别？

将来，我们可能会把 tulip.Future 和 concurrent.futures.Future 统一起来。例如，为后面个对象添加一个
__iter__() 方法，以支持 yield from 表达式。为了防止意外调用尚未完成的 result()
而阻塞事件循环，阻塞机制需要检测当前线程是否存在活动的事件循环，否则抛出异常。然而，这个PEP为了尽量减少外部依赖（只依赖
Python3.3），所以目前将不会对 concurrent.futures.Future 作出改变。

传输层

  

传输层是指基于 socket 或者其他类似的机制（例如，管道或者SSL连接）的抽象层。这里的传输层深受 Twisted 和 PEP 3153
的影响。用户很少会直接实现或者实例化传输层，事件循环提供了设置传输层的相关方法。

传输层是用来和协议一起工作的。典型的协议不关心底层传输层的具体细节，而传输层可以用来和多种的协议一起工作。例如，HTTP 客户端实现可以使用普通的
socket 传输层，也可以使用SSL传输层。普通 socket 传输层可以与 HTTP 协议外的大量协议一起工作（例如，SMTP, IMAP, POP,
FTP, IRC, SPDY)。

大多数连接有不对称特性：客户端和服务器通常有不同的角色和行为。因此，传输层和协议之间的接口也是不对称的。从协议的视角来看，发送数据通过调用传输层对象的
write() 方法来完成。write() 方法将数据放进缓冲区后立即返回。在读取数据时，传输层将充当一个更主动的角色：当从
socket（或者其他数据来源） 接到数据后，传输层将调用协议的 data_received() 方法。

传输层有以下公开方法：

write(data). 写数据。参数必须是一个 bytes 对象。返回 None。传输层可以自由缓存 bytes
数据，但是必须确保数据发送到另一端，并且维护数据流的行为。即：t.write(b'abc'); t.write(b'def') 等价于
t.write(b'abcdef')，也等价于：

t.write(b'a')

t.write(b'b')

t.write(b'c')

t.write(b'd')

t.write(b'e')

t.write(b'f')

writelines(iterable). 等价于：

for data in iterable:

   self.write(data)

write_eof(). 关闭写数据端的连接，将不再允许调用 write()
方法。当所有缓冲的数据传输之后，传输层将向另一端发信号，表示已经没有其他数据了。有些协议不支持此操作；那样的话，调用 write_eof()
将会抛出异常。（注意：这个方法以前叫做 half_close()，除非你明确知道具体含义，这个方法名字并不明确表示哪一端会被关闭。）

can_write_eof().  如果协议支持 write_eof()，返回 True ；否则返回 False。（当 write_eof()
不可用时，有些协议需要改变相应的行为，所以需要这个方法。例如，HTTP中，为了发送当前大小未知的数据，通常会使用 write_eof()
来表示数据发送完毕。但是，SSL
不支持这种行为，相应的HTTP协议实现需要使用分段（chunked）编码。但是如果数据大小在发送时未知，适用于两种情况的最好方案是使用 Content-
Length 头。）

pause(). 暂停发送数据，直接调用了 resume() 方法。在 pause() 调用，再到调用 resume() 之间，不会再调用协议的
data_received() 方法。在 write()  方法中无效。

resume(). 使用协议的 data_received() 重新开始传输数据。

close(). 关闭连接。在所有使用 write() 缓冲好的数据发送完毕之前，不会关闭连接。连接关闭后，协议的 data_received()
方法不会再被调用。当所有缓冲的数据发送完毕后，将会用 None 作为参数调用协议的 connection_lost()
方法。注意：这个方法不确保会调用上面所有的方法。

abort(). 中断连接。所有在缓冲区内尚未传输的数据都会被丢弃。不久后，协议的 connection_lost() 将会被调用，传入一个 None
参数。（待定：在 close(), abort() 或者另一端的关闭动作中，对 connection_lost() 传入不同的参数?
或者添加一个方法专门用来查询这个? Glyph 建议传入不同的异常）

尚未完成的：提供另一种流量控制的方法：传输层在缓冲区数据成为负担的情况下有可能暂停协议层。建议：如果协议有 pause() 和 resume()
方法的话，允许传输层调用它们；如果不存在，则协议不支持流量控制。（对 pause() 和 resume()，可能协议层和传输层使用不同的名称会好一点？）

协议 (Protocols)

  

协议通常和传输层一起配合使用。这里提供了几个常用的协议（例如，几个有用的HTTP客户端和服务器实现），大多数协议需要用户或者第三方库来实现。

一个协议必须实现以下的方法，这些方法将被传输层调用。这些回调函数将被事件循环在正确的上下文中调用（参考上面的上下文("Context")小节）。

connection_made(transport).
意味着传输层已经准备好且连接到一个实现的另一端。协议应该将传输层引用作为一个变量保存起来（这样后面就可以调用它的 write()
及其他方法），也可以在这时发送握手请求。

data_received(data). 传输层已经从读取了部分数据。参数是一个不为空的的 bytes 对象。这个参数的大小并没有明确的限制。
p.data_received(b'abcdef') 应该等价于下面的语句：

p.data_received(b'abc')

p.data_received(b'def')

eof_received(). 在另一端调用了 write_eof() 或者其他等价的方法时，这个方法将被调用。默认的实现将调用传输层的 close()
方法，close() 方法调用了协议中的 connection_lost() 方法。

connection_lost(exc). 传输层已经关闭或者中断了，另一端已经安全地关闭了连接，或者发生了异常。在前三种情况中，参数为
None；在发生了异常的情况下，参数是导致传输层中断的异常。（待定：是否需要区分对待前三种情况？）

这里是一个表示了调用的顺序和多样性的图:

connection_made()-- exactly once

data_received()-- zero or more times

eof_received()-- at most once

connection_lost()-- exactly once

补充: 讨论用户的代码是否要做一些事情保证协议和传输协议不会被过早地GC（垃圾回收）。

回调函数风格

  

大部分接口采取回调函数也采取位置参数。举例来说，要安排foor("abc",42)马上调用，你可以调用ev.call_soon(foo,"abc",42)。
要计划调用foo()，则使用ev.call_soon(foo)。这种约定大大地减少了需要典型的回调函数编程的小lambda表达式的数量。

这种约定明确的不支持关键字参数。关键字参数常用来传递可选的关于回调函数的额外信息。这允许API的优雅的改革，不用担心是否一个关键字在某些地方被一个调用者声明
。如果你有一个必需使用关键字参数调用的回调函数，你可以使用lambda表达式或者functools.partial。例如：

ev.call_soon(functools.partial(foo, "abc", repeat=42))

  

选择一种事件循环的实现方式

待完成。（关于使用select/poll/epoll，以及如何改变选择。属于事件循环策略）

协程和调度器

  

这是一个独立的顶层部分，因为它的状态与事件循环接口不同。协程是可选的，而且只用回调的方式来写代码也是很好的。另一方面，只有一种调度器/协程的API实现，如果
你选择了协程，那就是你唯一要用的。

协同程序

  

一个协同程序是遵循一下约定的生产者。为了良好的文档的目的，所有的协同程序应该用@tulip.coroutine来修饰，但这并没严格要求。

协同程序使用在 PEP 380 里介绍的yield from语法啦代替原始的yield语法。

这个“协同程序”的词和“生产者”的含义类似，是用来描述两个不同（尽管是有关系）的概念。

定义了协同程序的函数（使用tulip.coroutine修饰定义的一个函数）。如果要消除歧义的话，我们可以称这个函数为协同程序函数(coroutine
function)。

通过一个协同函数获得对象。这个对象代表了一个最终会完成的计算或者I/O操作（通常两者都有）。我们可以称这个对象为协同程序对象(coroutine
object)来消除歧义。

  

协程能做的事情：

结果= 从future使用yield-- 直到future完成，挂起协程，然后返回future的结果，或者抛出它要传递的异常。

结果 = 从coroutine使用yield--等待另外的协程产生结果 （或者抛出一个要传递的异常）。这协程的异常必须是对另外一个协同程序的调用。

返回结果-- 为使用yield from表达式等待结果的协程返回一个结果。

抛出异常-- 在协程里为使用yield from表达式等待的（程序）抛出一个异常。

调用一个协程不会马上执行它的代码——它仅仅是一个生产者，同时通过调用而返回的协程确实只是一个生产者对象，它在你迭代它之前不会做任何事情。对于协程来说，有两种
基本的方法来让它开始运行：从别的协程里调用yield（假设另外的协程已经在运行了！），或者把它转换为一个Task（看下面）。

协程只有在事件循环在运行时才能运行。

等待多个协同程序

  

有两个类似wait()和as_completed()，在包concurrent.futures里的API提供来等待多个协同程序或者Future：

tulip.wait(fs, timeout=None, return_when=ALL_COMPLETED)。 这是一个由fs提供等待Future或者其他
协同程序完成的协同程序。协同程序参数会封装在Task里（看下面）。这方法会返回一个Future，Future的成功结果是一个包含了两个Future集的元组（
做完(done)，挂起(pending)），done是一个原始Future（或封装过的协同程序）的集合表示完成（或取消），以及pending表示休息，例如还
没完成（或者取消）。可选参数timeout和return_when与concurrent.futures.wait()里的参数都有同样的意思和默认值:tim
eout，如果不是None，则为全部的操作指定一个timeout；return_when，指定什么时候停止。常量FIRST_COMPLETED，FIRST_
EXCEPTION，ALL_COMPLETED使用相同的值定义同时在 PEP 3148里有相同的意思：

ALL_COMPLETED(default): 等待，知道所有的Future处理了或者完成了 （或者直到超时发生）。

FIRST_COMPLETED: 等待，知道至少一个Future做好了或者取消（或者直到超时发生）。

FIRST_EXCEPTION: 等待，知道至少一个Future由于异常而做好（非取消） （这种从过滤器中排除取消的Future是很神奇的，但PEP
3148 就用这种方法里做了。）

tulip.as_completed(fs, timeout=None). 返回一个值为Future的迭代器;
等待成功的值，直到下一个Future或者协同程序从fs中完成都处于等待状态， 同时返回它自己的结果
（或者抛出它的异常）。可选参数timeout与concurrent.futures.wait()里的参数都有同样的意思和默认值: 当存在超时的时候，
下一个由迭代器返回的Future在等待的时候会抛出异常TimeoutError。 使用列子：

for f in as_completed(fs):

   result = yield from f  # May raise an exception.

   # Use result.

  

任务（Task）

  

任务（Task）是一个管理独立运行子程序的对象。Task接口和Future接口是一样的。如果子程序完成或者抛出异常，那么与之关联的任务也就完成了。返回的结果
就是相应任务的结果，抛出的异常就是相应任务的异常。

如果取消一个尚未完成的任务，将会阻止关联的子程序继续执行。在这种情况下，子程序将会接收到一个异常，以更好地处理这个取消命令。当然，子程序不一定要处理这个异常
。这种机制通过调用生成器标准的  close() 方法，这个在 PEP 342 中有描述。

任务对于子程序间的交互以及基于回调的框架（例如 Twisted）也很有用。将一个子程序转化成任务之后，就可以将回调函数加到任务里。

你可能会问，为什么不将所有子程序都转化成任务？ @tulip.coroutinedecorator
可以实现这个任务。这样的话，如果一个子程序调用了另外的子程序（或者其他复杂情况），那么整个程序将会变得相当慢。在复杂情况下，保持子程序比转换成任务要快得多。

调度器

  

调度器没有公开的接口。你可以使用 future 和 task 的 yield
和调度器进行交互。实际上，调度器并没有一个具体的类实现。通过使用事件循环的公共接口，Future 和 Task
类表现了调度器的行为。所以即使换了第三方的事件循环实现，调度器特性也可以使用。

睡眠

  

尚未完成的：yield sleep(秒). 可以用 sleep(0) 来挂起并查询I/O。

协同程序与协议

  

使用协同程序去实现协议的最好方法坑农是使用一个流缓存，这缓存使用data_received()来填充数据，同时能够使用像read(n)和readline()
等返回一个Future的方法来异步读取数据。当链接关闭时，read()方法应该返回一个Future，这Future的结果会是''，或者如果connectio
n_closed()由于异常而被调用，则抛出一个异常。

要写入数据的话，在transport上的write()（及同类型的）方法能够使用——这些不会返回一个Future。应该提供用于设置和在调用了connecti
on_made()时开始协同程序的一个标准的协议实现。

补充：更多的规范。

取消

  

补充。当一个任务被取消了，它的协同程序会在它从调度程序中放弃的任何一个地方中看到异常（例如可能在操作中放弃）。我们需要讲清楚要抛出什么异常。

再次补充：超时。

已知问题

  

调试的API？ 例如 一些能够记录大量材料的或者是记录不常用条件的 （就像队列填充比排空快）或者甚至回调函数耗费大量时间...

我们需要内省的API吗？例如请求读回调函数返回一个文件描述符。或者当下一个调度的（回调函数）被调用时。或者由回调函数注册了的一些文件描述符。

传输可能需要一个方法来试着返回socket的地址（和另外的方法返回对等的地址）。尽管这是依赖于socket的类型，同时这并不总是一个socket；然后就应该
返回None。（作为选择，可以有个方法来返回socket自身——但可以想到一个没有使用socket来实现IP链接，那它应该怎么做呢？）

需要处理os.fokd()。（这可能在Tulip的情况下会上升到选择器类。）

也许start_serving()需要一个方法来传入到一个当前的socket（例如gunicorn就需要这个）。create_connection()也有同
样的问题。

我们也许会介绍一些明确的锁，尽管使用起来有点痛苦，因为我们不能使用with锁：阻塞语法（因为要等待一个锁的话，我们就要用yield
from了，这是with语句不能做到的）。

是否要支持数据报协议、链接。可能要更多的套接字I/O方法，例如sock_sendto()和sock_recvfrom()。或者用户客户自己编写（这不是火箭科
学）。有它的理由去覆盖write()，writelines()，data_received()到一个单一的数据报？（Glyph推荐后者。）然后用什么来代替w
rite()？最后，我们需要支持无连接数据报协议吗？（这意味着要封装sendto()和recvfrom()。）

我们可能需要API来控制各种超市。例如我们可能想限制在解析DNS、链接、SSL握手、空闲链接、甚至是每个会话里消耗的时间。也许有能充分的加入timeout的
关键字参数到一些方法里，和其他能够巧妙地通过call_later和Task.cancel()来实现超时。但可能有些方法需要默认的超时时间，同时我们可能想改变
这种规范的全局操作的默认值。（例如，每个事件循环）。

一个NodeJS风格的事件触发器？或者把这个作为一个单独的教程？这其实在用户空间做是足够容易的，尽管可能放在标准化里好一点（看https://github.
com/mnot/thor/blob/master/thor/events.py和https://github.com/mnot/thor/blob/mas
ter/doc/events.md 举的例子。）

  

参考文献

  

PEP 380 来自于TBD：Greg Ewing的教程描述yield的语义。

PEP 3148 描述concurrent.futures.Future.

PEP 3153,虽然被拒绝了, 但很好的描述了分离传输和协议的需要。

Tulip repo: http://code.google.com/p/tulip/

Nick Coghlan在一些背景下写的一个很好的博客条目，关于异步I/O不同处理、gevent、以及怎样使用future就像wihle、for和with的
概念的思想， : http://python-
notes.boredomandlaziness.org/en/latest/pep_ideas/async_programming.html

TBD: 有关Twisted、Tornado、ZeroMQ、pyftpdlib、libevent、libev、pyev、libuv、wattle等的参考

鸣谢

  

除了PEP 3153之外, 受到的影响包括 PEP 380 and Greg Ewing的yield from的教程, Twisted, Tornado,
ZeroMQ, pyftpdlib, tulip (作者的企图是想把这些全部综合起来), wattle (Steve Dower的相反的提议),
2012年9月到12月在python-ideas上大量讨论, 一个与Steve Dower和 Dino Viehland在Skype上的会话, 和Ben
Darnell的电子邮件交流, 一个Niels Provos的听众 (libevent原始作者),两场和几个Twisted开发者的面对面会议， 包括了
Glyph、Brian Warner、David Reid、 以及Duncan McGreggor。同样的，作者前期在为Google App
Engine的NDB库的异步支持也有重要的影响。

  

