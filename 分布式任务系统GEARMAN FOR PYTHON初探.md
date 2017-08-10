# 分布式任务系统GEARMAN FOR PYTHON初探

了解Gearman，请访问gearman官网：http://gearman.org/index.php?id=getting_started

Gearman for Python API Doc: http://pythonhosted.org/gearman/

  

++++++++++++++++++++++++++++++++++++++++++++

安装Gearman:

++++++++++++++++++++++++++++++++++++++++++++

基础依赖库：

    
    
    yum install boost-devel libevent-devel sqlite-devel libuuid-devel
    wget https://launchpad.net/gearmand/trunk/0.33/+download/gearmand-0.33.tar.gz  
    tar xzvf gearmand-0.33.tar.gz  
    cd gearmand-0.33  
    ./configure 
    make 
    make install

++++++++++++++++++++++++++++++++++++++++++++

安装Gearman Python客户端

++++++++++++++++++++++++++++++++++++++++++++

    
    
    wget http://pypi.python.org/packages/source/g/gearman/gearman-2.0.2.tar.gz#md5=3847f15b763dc680bc672a610b77c7a7  
        tar xvzf  gearman-2.0.2.tar.gz  
        python setup.py install 

获取直接用自动安装： easy_install gearman

启动服务：gearmand -d  

启动Worker：gearman -w -f wc -- wc -l &

-w 代表启动的是worker，-f wc 代表启动一个task名字为wc， -- wc -l表示这个task是做wc -l 统计行数。

启动Client：gearman -f wc < /etc/passwd

++++++++++++++++++++++++++++++++++++++++++++

python work代码：

++++++++++++++++++++++++++++++++++++++++++++

    
    
    import os  
    import gearman  
    import math        
    class MyGearmanWorker(gearman.GearmanWorker):    
        def on_job_execute(self, current_job):    
            print "Job started"  
            return super(MyGearmanWorker, self).on_job_execute(current_job)    
        
    def task_callback(gearman_worker, gearman_job):    
        print gearman_job.data   
        return gearman_job.data  
        
    my_worker = MyGearmanWorker(['192.168.0.75:4730'])    
    my_worker.register_task("echo", task_callback)    
    my_worker.work()

++++++++++++++++++++++++++++++++++++++++++++

python client代码：

++++++++++++++++++++++++++++++++++++++++++++

    
    
    from gearman import GearmanClient        
    gearman_client = GearmanClient(['192.168.0.75:4730'])  
    gearman_request = gearman_client.submit_job('echo', 'foo')  
    result_data = gearman_request.result  
    print result_data

  

