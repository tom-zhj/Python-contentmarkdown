# python中smtplib使用注意点

使用smtplib时，打开的server，最好使用quit方法来关闭连接，而不是close。

    
    
    server.quit() #好
    #server.close() #不好

因为quit不仅仅会关闭连接，还会关闭session。这个session会跨越连接，而且当这个session中有退信发生时，后续发出的信件会爆出奇怪的SMT
P协议错误。

使用smtplib时，即便每次都重新open server，对dns的解析也只有一次，这样当一个域名下有多个smtp
server本来可以用于负载均衡的环境下，使用smtplib的python程序就总是使用一台机器，没法负载均衡，影响了伸缩性。为此，想到的办法是
单独对邮件服务器域名进行解析，得到所有的机器名，然后随机选一台smtp
server来连接，做一个应用层的负载均衡。可以考虑使用下面这段代码，感谢茂兴的提供：

    
    
    class smtp_server_factory(object):
        def _get_addr_from_name(self, hostname):
            addrs = socket.getaddrinfo(hostname, smtplib.SMTP_PORT, 0, socket.SOCK_STREAM)
            return [addr[4][0] for addr in addrs]
     
        def get_server(self, hostname):
            addrs = self._get_addr_from_name(hostname)
            random.shuffle(addrs)
            for addr in addrs:
                try:
                    smtp_server = smtplib.SMTP(addr)
                except Exception, e:
                    pass
                else:
                    print addr
                    return smtp_server
            return None

#使用

    
    
    server=smtp_server_factory().get_server('xxx-mail.qq.com')

  

