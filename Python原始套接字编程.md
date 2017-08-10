# Python原始套接字编程

在实验中需要自己构造单独的HTTP数据报文，而使用SOCK_STREAM进行发送数据包，需要进行完整的TCP交互。

因此想使用原始套接字进行编程，直接构造数据包，并在IP层进行发送，即采用SOCK_RAW进行数据发送。

使用SOCK_RAW的优势是，可以对数据包进行完整的修改，可以处理IP层上的所有数据包，对各字段进行修改，而不受UDP和TCP的限制。

下面开始构造HTTP数据包，

IP层和TCP层使用python的Impacket库，http内容自行填写。

    
    
    #!/usr/bin/env python
     
    #-------------------------------------------------------------------------------
    # Name:     raw_http.py
    # Purpose:       construct a raw http get packet
    #
    # Licence:   PythonTab.com    
    #-------------------------------------------------------------------------------
     
    import sys
    import socket
    from impacket import ImpactDecoder, ImpactPacket
     
    def main():
     
        if len(sys.argv) = 1:
            # Calculate its checksum.
            seq_id = seq_id + 1
            tcp.set_th_seq(seq_id)
            tcp.calculate_checksum()
     
            # Send it to the target host.
            s.sendto(ip.get_packet(), (dst,80))
            cnt= cnt -1
     
    if __name__ == '__main__':
        main()

  

