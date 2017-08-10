# 使用python管理Cisco设备

今天发现一个老外使用python写的管理cisco设备的小框架[tratto](https://github.com/akonkol/tratto)，可以用
来批量执行命令。  

下载后主要有3个文件：

Systems.py 定义了一些不同设备的操作系统及其常见命令。

Connectivity.py 是主要实现功能的代码，其实主要就是使用了python的pexpect模块。

Driver.py是一个示例文件。

[root@safe tratto-master]# cat driver.py

    
    
    #!/usr/bin/env python
    import Connectivity
    import Systems 
    #telnet to a cisco switch
    m = Systems.OperatingSystems['IOS']
    s = Connectivity.Session("192.168.1.1",23,"telnet",m)
    s.login("yourusername", "yourpassword")
    # if your need to issue an "enable" command
    s.escalateprivileges('yourenablepassword')
    s.sendcommand("show clock")
    s.sendcommand("show run")
    s.logout()

以上就是示例driver.py的内容，使用很简单。

首先选择一个设备系统版本，此例cisco交换机，所以使用了IOS。作者现在写的可以支持的设备系统有：

OperatingSystems = {

   'IOS': CiscoIOS,

   'WebNS': CiscoWebNS,

   'OSX': AppleOSX,

   'SOS': SecureComputingSidewinder,

   'AOS': ArubaOS,

   'OBSD': OpenBSD,

   }

然后填写ip，端口，telnet或者ssh，最后就是上步选择的系统版本。login填上登陆凭证。

s.escalateprivileges是特权凭证。so easy~

以下是我写的一个使用脚本，抓取交换机的一些信息，然后保存到文件。

[root@safe tratto-master]# cat  cisco.py

    
    
    #!/usr/bin/env python
    #
    # Cisco Switch commands 
    # By s7eph4ni3
    #
    import Connectivity
    import Systems 
    m = Systems.OperatingSystems['IOS']
    iplist = ['192.168.1.1','192.168.1.2']
    cmdlist = ['show ip int brief','show cdp nei detail','show arp','show ver']
    for ip in iplist:
        if ip == '192.168.1.1':
            s = Connectivity.Session(ip,23,"telnet",m)
            s.login("", "passwd")
        else:
            s = Connectivity.Session(ip,22,"ssh",m)
            s.login("username", "passwd")
        s.escalateprivileges('enpasswd')
        f = open(ip+'.txt','w+')
        for cmd in cmdlist:
            a = s.sendcommand(cmd)
            f.write(ip+cmd+'\n')
            f.write(a+'\n')
        f.close()
        s.logout()

  

  

