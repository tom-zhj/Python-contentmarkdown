# Python获取服务器的厂商和型号信息

Python获取服务器的厂商和型号信息，在RHEHL6下，需要系统预装python-dmidecode这个包（貌似默认就已经装过了）

脚本内容如下

[root@linuxidc tmp]# cat test.py

    
    
    #!/usr/bin/env python
    import dmidecode
    info=dmidecode.system()
    info_keys=info.keys()
    for i in range(len(info_keys)):
        if info[info_keys[i]]['dmi_type'] == 1 :
            print info[info_keys[i]]['data']['Manufacturer']
            print info[info_keys[i]]['data']['Product Name']

[root@linuxidc tmp]#

执行的时候，需要root权限，输出如下：

[root@linuxidc tmp]# ./test.py

    
    
    HP
    ProLiant DL380p Gen8

第一行是厂商HP，第二行是HP服务器的型号。



注：通过dmidecode命令获取这些信息的方式是：

    
    
    dmidecode -t1

输出如下：

[root@linuxidc tmp]# dmidecode -t1

    
    
    # dmidecode 2.11
    SMBIOS 2.7 present.
     
    Handle 0x0100, DMI type 1, 27 bytes
    System Information
            Manufacturer: HP
            Product Name: ProLiant DL380p Gen8
            Version: Not Specified
            Serial Number: CNG230SHDQ
            UUID: 32333536-3030-4E43-4732-333053484451
            Wake-up Type: Power Switch
            SKU Number: 653200-B21
            Family: ProLiant

[root@linuxidc tmp]#

  

