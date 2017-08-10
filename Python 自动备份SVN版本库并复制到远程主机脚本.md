# Python 自动备份SVN版本库并复制到远程主机脚本


    #!/usr/bin/python  
    # -*- coding: utf-8 -*-  
      
    import os 
    import re 
    import tarfile 
    import datetime 
    import pexpect 
    basedir='/data/bak/' #文件夹  
    iplist=['']# IP地址  
    def get_list(txt_file): 
        ret_list = [] 
        fin = open(txt_file,'r') 
        for line in fin: 
            if (re.match('^\\s*$',line)):     #跳过是空白的行   
                continue
            else: 
                line = line.lstrip() 
                line = line.rstrip()  #将回车（\n）去掉  
                ret_list.append(line) 
        #print('debug info of get_list :\n',ret_list)  
        return ret_list 
          
    def copy_svn(filelist): #这个函数主要是完成dump  
        name= [] 
        name=os.path.split(filelist) 
        now = datetime.datetime.now() 
        filename = now.strftime(basedir+iplist[0] +name[-1]+ '_%Y%m%d_%H%M%S.dump') 
        os.system('svnadmin dump ' + filelist +' > '+filename ) 
        
        tarname =  now.strftime(basedir+iplist[0]+'_SVNDump_' +name[-1]+ '_%Y%m%d_%H%M%S.tar.gz') 
        #print tarname  
        tar = tarfile.open(tarname, 'w|gz') 
        tar.add(filename) 
        tar.close() 
          
          
        scp = pexpect.spawn('scp -r ' +  tarname  + ' root@IP:/data/databak/FilesBack/') 
        scp.expect('.ssword:*') 
        scp.sendline('密码')   
        scp.expect(pexpect.EOF, timeout=None) 
          
        
        olddate = (now - datetime.timedelta(5)).strftime("%Y%m%d") 
        print olddate 
             
        for i in os.listdir(basedir): 
            file = re.search(r'\w*[_](\d{8})[_]\d{6}.(tar.gz|dump)', i) 
            #print i, file  
            if file and olddate>=file.group(1): 
                os.remove(basedir + file.group(0)) 
                print 'del:', file.group(0) 
                filelog=open("/data/bak/bak.log", "a") 
                filelog.write("============DATE:%s============= \n"% now.strftime("%Y%m%d")) 
                filelog.write("del file:%s \n" % (basedir+file.group(0) )) 
                filelog.write("============DATE:%s============= \n"% now.strftime("%Y%m%d")) 
            filelog.close() 
         
    def copy_files(txt_file): 
        geted_list = get_list(txt_file) 
        for file in geted_list: 
            copy_svn(file) 
          
    if __name__ == '__main__': 
        copy_files('/data/bak/filebak.txt') 
        print '='*20,'\ncopy_OKOKOK\n','='*20 
     
    #!/usr/bin/python
    # -*- coding: utf-8 -*-
     
    import os
    import re
    import tarfile
    import datetime
    import pexpect
    basedir='/data/bak/' #文件夹
    iplist=['']# IP地址
    def get_list(txt_file):
        ret_list = []
        fin = open(txt_file,'r')
        for line in fin:
            if (re.match('^\\s*$',line)):     #跳过是空白的行
                continue
            else:
                line = line.lstrip()
                line = line.rstrip()  #将回车（\n）去掉
                ret_list.append(line)
        #print('debug info of get_list :\n',ret_list)
        return ret_list
        
    def copy_svn(filelist): #这个函数主要是完成dump
        name= []
        name=os.path.split(filelist)
        now = datetime.datetime.now()
        filename = now.strftime(basedir+iplist[0] +name[-1]+ '_%Y%m%d_%H%M%S.dump')
        os.system('svnadmin dump ' + filelist +' > '+filename )
      
        tarname =  now.strftime(basedir+iplist[0]+'_SVNDump_' +name[-1]+ '_%Y%m%d_%H%M%S.tar.gz')
        #print tarname
        tar = tarfile.open(tarname, 'w|gz')
        tar.add(filename)
        tar.close()
        
        
        scp = pexpect.spawn('scp -r ' +  tarname  + ' root@IP:/data/databak/FilesBack/')
        scp.expect('.ssword:*')
        scp.sendline('密码') 
        scp.expect(pexpect.EOF, timeout=None)
        
      
        olddate = (now - datetime.timedelta(5)).strftime("%Y%m%d")
        print olddate
           
        for i in os.listdir(basedir):
            file = re.search(r'\w*[_](\d{8})[_]\d{6}.(tar.gz|dump)', i)
            #print i, file
            if file and olddate>=file.group(1):
                os.remove(basedir + file.group(0))
                print 'del:', file.group(0)
               filelog=open("/data/bak/bak.log", "a")
                filelog.write("============DATE:%s============= \n"% now.strftime("%Y%m%d"))
                filelog.write("del file:%s \n" % (basedir+file.group(0) ))
                filelog.write("============DATE:%s============= \n"% now.strftime("%Y%m%d"))
         filelog.close()
       
    def copy_files(txt_file):
        geted_list = get_list(txt_file)
        for file in geted_list:
            copy_svn(file)
        
    if __name__ == '__main__':
        copy_files('/data/bak/filebak.txt')
        print '='*20,'\ncopy_OKOKOK\n','='*20

  

