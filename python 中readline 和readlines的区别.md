# python 中readline 和readlines的区别

#### 1.readline 场景

    
    
    f0=file("readline.txt",r)
    while true
        for line in f0.readline()
    if not line: break
    pass #do something

readline 的用法，速度是fileinput的3倍左右，每秒3-4万行，好处是 一行行读 ，不占内存，适合处理比较大的文件，比如超过内存大小的文件

#### 2.readlines 场景

    
    
    f1=open("readline.txt","r")
    for line in f1.readlines()#跟上面的方式不同
    print line

readlines会把文件都读入内存，速度大大增加，但是木有这么大内存,那就只能乖乖的用readline了

  

