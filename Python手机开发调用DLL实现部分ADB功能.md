# Python手机开发调用DLL实现部分ADB功能

近期学了一点Python，然后正好有一个手机同步工具方面的预研工作要完成。

要实现PC与手机的通信，首先要找到他们的通信协议，还好的是Android有完善的协议：ADB

ADB的代码是开源的，而且支持Windows平台，有现成的DLL可以调用：AdbWinApi.dll，AdbWinUsbApi.dll

好了，可以用VC搞定，但我想用Python试一下，于是开始了苦逼的查资料+实验的过程。

实验过程就不多说了，由于上面的两个DLL都是用C实现的，提供的头文件也是C语言的，所以有了下面这个python测试程序（Python2.7）：

    
    
    import ctypes 
      
    #自定义的GUID结构，有兴趣的可以自己研究用uuid模块  
    class GUID(ctypes.Structure): 
        _fields_ = [("Data1", ctypes.c_ulong), 
                    ("Data2", ctypes.c_ushort), 
                    ("Data3", ctypes.c_ushort), 
                    ("Data4", ctypes.c_ubyte*8)] 
      
    #自己定义的一个结构体，便于使用DLL接口  
    class AdbInterfaceInfo(ctypes.Structure): 
        _fields_ = [("class_id", GUID), 
                    ("flags", ctypes.c_ulong), 
                    ("device_name", ctypes.c_wchar*800)] 
      
    def strGUID(GUID): 
        string = '' 
        string = string + '%x' % buff.class_id.Data1 + '-%x' % buff.class_id.Data2 + '-%x' % buff.class_id.Data3 
        string = string + '-%x' % buff.class_id.Data4[0] 
        string = string + '%x' % buff.class_id.Data4[1] 
        string = string + '%x' % buff.class_id.Data4[2] 
        string = string + '%x' % buff.class_id.Data4[3] 
        string = string + '%x' % buff.class_id.Data4[4] 
        string = string + '%x' % buff.class_id.Data4[5] 
        string = string + '%x' % buff.class_id.Data4[6] 
        string = string + '%x' % buff.class_id.Data4[7] 
        return string 
      
    dll = ctypes.cdll.LoadLibrary('AdbWinApi.dll') 
    usb_class_id = GUID(0xF72FE0D4, 0xCBCB, 0x407d, (0x88, 0x14, 0x9e, 0xd6, 0x73, 0xd0, 0xdd, 0x6b)) 
    enum_handle = dll.AdbEnumInterfaces(usb_class_id, ctypes.c_bool('true'), ctypes.c_bool('true'), ctypes.c_bool('true')) 
      
    while(1): 
        buff = AdbInterfaceInfo() 
        size = ctypes.c_ulong(ctypes.sizeof(buff)) 
        status = dll.AdbNextInterface(enum_handle, ctypes.byref(buff), ctypes.byref(size)) 
      
        if status==1: 
            #print "GUID = " + strGUID(buff.class_id)  
            #print "status = " + str(status)  
            #print "Name = " + str(buff.device_name)  
            hAdbApi = dll.AdbCreateInterfaceByName(buff.device_name); 
            if hAdbApi == 0: 
                print 'AdbCreateInterfaceByName Fail'
            else: 
                serial = ' '*128
                pserial = ctypes.c_char_p() 
                pserial.value = serial 
                serial_len = ctypes.c_ulong(len(serial)) 
                ret = dll.AdbGetSerialNumber(hAdbApi, pserial, ctypes.byref(serial_len), ctypes.c_bool('false')); 
                if ret == 1: 
                    print 'Device Name: ' + '%s'% serial 
                else: 
                    print 'Get Device Name Fail'
        else: 
            print 'Finished'
            break
    import ctypes
    #自定义的GUID结构，有兴趣的可以自己研究用uuid模块
    class GUID(ctypes.Structure):
        _fields_ = [("Data1", ctypes.c_ulong),
                    ("Data2", ctypes.c_ushort),
                    ("Data3", ctypes.c_ushort),
                    ("Data4", ctypes.c_ubyte*8)]
    #自己定义的一个结构体，便于使用DLL接口
    class AdbInterfaceInfo(ctypes.Structure):
        _fields_ = [("class_id", GUID),
                    ("flags", ctypes.c_ulong),
                    ("device_name", ctypes.c_wchar*800)]
    def strGUID(GUID):
        string = ''
        string = string + '%x' % buff.class_id.Data1 + '-%x' % buff.class_id.Data2 + '-%x' % buff.class_id.Data3
        string = string + '-%x' % buff.class_id.Data4[0]
        string = string + '%x' % buff.class_id.Data4[1]
        string = string + '%x' % buff.class_id.Data4[2]
        string = string + '%x' % buff.class_id.Data4[3]
        string = string + '%x' % buff.class_id.Data4[4]
        string = string + '%x' % buff.class_id.Data4[5]
        string = string + '%x' % buff.class_id.Data4[6]
        string = string + '%x' % buff.class_id.Data4[7]
        return string
    dll = ctypes.cdll.LoadLibrary('AdbWinApi.dll')
    usb_class_id = GUID(0xF72FE0D4, 0xCBCB, 0x407d, (0x88, 0x14, 0x9e, 0xd6, 0x73, 0xd0, 0xdd, 0x6b))
    enum_handle = dll.AdbEnumInterfaces(usb_class_id, ctypes.c_bool('true'), ctypes.c_bool('true'), ctypes.c_bool('true'))
    while(1):
        buff = AdbInterfaceInfo()
        size = ctypes.c_ulong(ctypes.sizeof(buff))
        status = dll.AdbNextInterface(enum_handle, ctypes.byref(buff), ctypes.byref(size))
        if status==1:
            #print "GUID = " + strGUID(buff.class_id)
            #print "status = " + str(status)
            #print "Name = " + str(buff.device_name)
            hAdbApi = dll.AdbCreateInterfaceByName(buff.device_name);
            if hAdbApi == 0:
                print 'AdbCreateInterfaceByName Fail'
            else:
                serial = ' '*128
                pserial = ctypes.c_char_p()
                pserial.value = serial
                serial_len = ctypes.c_ulong(len(serial))
                ret = dll.AdbGetSerialNumber(hAdbApi, pserial, ctypes.byref(serial_len), ctypes.c_bool('false'));
                if ret == 1:
                    print 'Device Name: ' + '%s'% serial
                else:
                    print 'Get Device Name Fail'
        else:
            print 'Finished'
            break

  

上面这个简单的Python代码，可以通过AdbWinApi.dll和AdbWinUsbApi.dll这两个DLL来找到你PC上正连接着的Android设备。

只调用了3个DLL接口，但目的已经达到，可以得出下面的结论：

使用Python调用DLL的方式来实现ADB工具是可行的，当然麻烦是不会少的了。

写在最后，Python调用C写的DLL还是比较麻烦的，特别是参数传递，尤其是指针的处理，这方面还得依靠ctypes模块。。。

  

