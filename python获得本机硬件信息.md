# python获得本机硬件信息

注意：这段代码需要wmi  和 系统 win32 扩展支持。

没安装库的要先下载安装，我装的是 WMI-1.4.6.win32 和 pywin32-218.win32-py2.7

还有，代码里面文件目录自己修改下咯。

    
    
    # -*- coding:gb2312 -*-  
    import wmi
    hardware=file('F:\Python\Hardware.txt','w')
     
    w=wmi.WMI()
    hardware.write("cpu型号，主频：\n")
    for processor in w.Win32_Processor():          
    hardware.write("Processor ID: %s" % processor.DeviceID)
    hardware.write("\nProcess Name: %s" % processor.Name.strip()+'\n\n')
    hardware.write('内存大小：')
    totalMemSize=0
    for memModule in w.Win32_PhysicalMemory():   
    totalMemSize+=int(memModule.Capacity)
    hardware.write("\nMemory Capacity: %.2fMB" %((totalMemSize+1048575)/1048576)+'\n\n')
    hardware.write('硬盘使用情况：')
    for disk in w.Win32_LogicalDisk (DriveType=3):
    temp=disk.Caption+" %0.2f%% free" %(100.0 * long (disk.FreeSpace) / long (disk.Size))
    hardware.write('\n'+temp)
    hardware.write('\n')
    hardware.write('\n显示IP和MAC：\n')
    for interface in w.Win32_NetworkAdapterConfiguration (IPEnabled=1):
    hardware.write('网卡驱动信息：')
    hardware.write(interface.Description+'\n')
    hardware.write('网卡MAC地址：')
    hardware.write(interface.MACAddress+'\n')
    #for ip_address in interface.IPAddress:
    hardware.write('IP地址：')
    hardware.write(interface.IPAddress[0]+'\n')
    hardware.write('外网IP接口')
    hardware.write(interface.IPAddress[1]+'\n')
    hardware.close()

运行效果图

![](http://www.pythontab.com/uploadfile/2013/0416/20130416094454933.png)  

  

