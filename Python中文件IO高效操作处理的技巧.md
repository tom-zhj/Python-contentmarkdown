# Python中文件I/O高效操作处理的技巧

### 如何读写文本文件？

实际案例

某文本文件编码格式已直(如UTF-8,GBK,BIG5)，在python2.x和python3.x中分别如何读取这些文件？

解决方案

注意区分python2和python3中的差别

字符串的语义发生了变化：

python2

python3

str

bytes

unicode

str

  
python2.x 写入文件前对 unicode 编码，读入文件后对二进制字符串解码

    
    
    >>> f = open('py2.txt', 'w')
    >>> s = u'你好'
    >>> f.write(s.encode('gbk'))
    >>> f.close()
    >>> f = open('py2.txt', 'r')
    >>> t = f.read()
    >>> print t.decode('gbk')

你好

python3.x 中 open 函数指定 t 的文本模式， encoding 指定编码格式

    
    
    >>> f = open('py3.txt', 'wt', encoding='utf-8')
    >>> f.write('你好')
    2
    >>> f.close()
    >>> f = open('py3.txt', 'rt', encoding='utf-8')
    >>> s = f.read()
    >>> s
    '你好'

### 如何设置文件的缓冲

实际案例

将文件内容写入到硬盘设备时，使用系统调用，这类I/O操作的时间很长，为了减少I/O操作的次数，文件通常使用缓冲区（有足够多的数据才进行系统调用），文件的缓存
行为，分为全缓冲、行缓存、无缓冲。

如何设置Python中文件对象的缓冲行文？

解决方案

全缓冲： open 函数的 buffering 设置为大于1的整数n，n为缓冲区大小

    
    
    >>> f = open('demo2.txt', 'w', buffering=2048)
    >>> f.write('+' * 1024)
    >>> f.write('+' * 1023)
    # 大于2048的时候就写入文件
    >>> f.write('-' * 2)
    >>> f.close()

行缓冲： open 函数的 buffering 设置为1

    
    
    >>> f = open('demo3.txt', 'w', buffering=1)
    >>> f.write('abcd')
    >>> f.write('1234')
    # 只要加上\n就写入文件中
    >>> f.write('\n')
    >>> f.close()

无缓冲： open 函数的 buffering 设置为0

    
    
    >>> f = open('demo4.txt', 'w', buffering=0)
    >>> f.write('a')
    >>> f.write('b')
    >>> f.close()

### 如何将文件映射到内存？

实际案例

在访问某些二进制文件时，希望能把文件映射到内存中，可以实现随机访问.(framebuffer设备文件)

某些嵌入式设备，寄存器呗编址到内存地址空间，我们可以映射 /dev/mem 某范围，去访问这些寄存器

如果多个进程映射到同一个文件，还能实现进程通信的目的

解决方案

使用标准库中的 mmap 模块的 mmap() 函数，它需要一个打开的文件描述符作为参数

创建如下文件

    
    
    [root@pythontab.com ~]# dd if=/dev/zero of=demo.bin bs=1024 count=1024
    1024+0 records in
    1024+0 records out
    1048576 bytes (1.0 MB) copied, 0.00380084 s, 276 MB/s
    # 以十六进制格式查看文件内容
    [root@pythontab.com ~]# od -x demo.bin 
    0000000 0000 0000 0000 0000 0000 0000 0000 0000
    *
    4000000
    
    
    >>> import mmap
    >>> import os
    >>> f = open('demo.bin','r+b')
    # 获取文件描述符
    >>> f.fileno()
    3
    >>> m = mmap.mmap(f.fileno(),0,access=mmap.ACCESS_WRITE)
    >>> type(m)
    <type 'mmap.mmap'>
    # 可以通过索引获取内容
    >>> m[0]
    '\x00'
    >>> m[10:20]
    '\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00'
    # 修改内容
    >>> m[0] = '\x88'

查看

    
    
    [root@pythontab.com ~]# od -x demo.bin 
    0000000 0088 0000 0000 0000 0000 0000 0000 0000
    0000020 0000 0000 0000 0000 0000 0000 0000 0000
    *
    4000000

修改切片

    
    
    >>> m[4:8] = '\xff' * 4

查看

    
    
    [root@pythontab.com ~]# od -x demo.bin 
    0000000 0088 0000 ffff ffff 0000 0000 0000 0000
    0000020 0000 0000 0000 0000 0000 0000 0000 0000
    *
    4000000
    
    
    >>> m = mmap.mmap(f.fileno(),mmap.PAGESIZE * 8,access=mmap.ACCESS_WRITE,offset=mmap.PAGESIZE * 4) 
    >>> m[:0x1000] = '\xaa' * 0x1000

查看

    
    
    [root@pythontab.com ~]# od -x demo.bin 
    0000000 0088 0000 ffff ffff 0000 0000 0000 0000
    0000020 0000 0000 0000 0000 0000 0000 0000 0000
    *
    0040000 aaaa aaaa aaaa aaaa aaaa aaaa aaaa aaaa
    *
    0050000 0000 0000 0000 0000 0000 0000 0000 0000
    *
    4000000

### 如何访问文件的状态？

实际案例

在某些项目中，我们需要获得文件状态，例如：

文件的类型(普通文件、目录、符号链接、设备文件…)

文件的访问权限

文件的最后的访问/修改/节点状态更改时间

普通文件的大小

…..

解决方案

当前目录有如下文件

    
    
    [root@pythontab.com 2017]# ll
    total 4
    drwxr-xr-x 2 root root 4096 Sep 16 11:35 dirs
    -rw-r--r-- 1 root root 0 Sep 16 11:35 files
    lrwxrwxrwx 1 root root 37 Sep 16 11:36 lockfile -> /tmp/qtsingleapp-aegisG-46d2-lockfile

系统调用

标准库中的os模块下的三个系统调用 stat 、 fstat 、 lstat 获取文件状态

    
    
    >>> import os
    >>> s = os.stat('files')
    >>> s
    posix.stat_result(st_mode=33188, st_ino=267646, st_dev=51713L, st_nlink=1, st_uid=0, st_gid=0, st_size=0, st_atime=1486197100, st_mtime=1486197100, st_ctime=1486197100)
    >>> s.st_mode
    33188
    >>> import stat
    # stat有很多S_IS..方法来判断文件的类型
    >>> stat.S_ISDIR(s.st_mode)
    False
    # 普通文件
    >>> stat.S_ISREG(s.st_mode)
    True

获取文件的访问权限，只要大于0就为真

    
    
    >>> s.st_mode & stat.S_IRUSR
    256
    >>> s.st_mode & stat.S_IXGRP
    0
    >>> s.st_mode & stat.S_IXOTH
    0

获取文件的修改时间

    
    
    # 访问时间
    >>> s.st_atime
    1486197100.3384446
    # 修改时间
    >>> s.st_mtime
    1486197100.3384446
    # 状态更新时间
    >>> s.st_ctime
    1486197100.3384446

将获取到的时间戳进行转换

    
    
    >>> import time
    >>> time.localtime(s.st_atime)
    time.struct_time(tm_year=2016, tm_mon=9, tm_mday=16, tm_hour=11, tm_min=35, tm_sec=47, tm_wday=4, tm_yday=260, tm_isdst=0)

获取普通文件的大小

    
    
    >>> s.st_size
    0

快捷函数

标准库中 os.path 下的一些函数，使用起来更加简洁

文件类型判断

    
    
    >>> os.path.isdir('dirs') 
    True
    >>> os.path.islink('lockfile')
    True
    >>> os.path.isfile('files') 
    True

文件三个时间

    
    
    >>> os.path.getatime('files')
    1486197100.3384445
    >>> os.path.getmtime('files')
    1486197100.3384445
    >>> os.path.getctime('files')
    1486197100.3384445

获取文件大小

    
    
    >>> os.path.getsize('files') 
    0

### 如何使用临时文件？

实际案例

某项目中，我们从传感器采集数据，每收集到1G数据后，做数据分析，最终只保存分析结果，这样很大的临时数据如果常驻内存，将消耗大量内存资源，我们可以使用临时文件
存储这些临时数据(外部存储)

临时文件不用命名，且关闭后会自动被删除

解决方案

使用标准库中的 tempfile 下的 TemporaryFile, NamedTemporaryFile

    
    
    >>> from tempfile import TemporaryFile, NamedTemporaryFile
    # 访问的时候只能通过对象f来进行访问
    >>> f = TemporaryFile()
    >>> f.write('abcdef' * 100000)
    # 访问临时数据
    >>> f.seek(0)
    >>> f.read(100)
    'abcdefabcdefabcdefabcdefabcdefabcdefabcdefabcdefabcdefabcdefabcdefabcdefabcdefabcdefabcdefabcdefabcd'
    >>> ntf = NamedTemporaryFile()
    # 如果要让每次创建NamedTemporaryFile()对象时不删除文件，可以设置NamedTemporaryFile(delete=False)
    >>> ntf.name
    # 返回当前临时文件在文件系统中的路径
    '/tmp/tmppNvNA6'

  

