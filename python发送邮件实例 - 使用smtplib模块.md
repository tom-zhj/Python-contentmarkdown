# python发送邮件实例 - 使用smtplib模块


    # 导入 smtplib 和 MIMEText 
    import smtplib 
    from email.mime.text import MIMEText 
       
    # 定义发送列表 
    mailto_list=["root@pythontab.com","10118157@qq.com"] 
       
    # 设置服务器名称、用户名、密码以及邮件后缀 
    mail_host = "smtp.163.com"
    mail_user = "xx@163.com"
    mail_pass = "xx"
    mail_postfix="163.com"
       
    # 发送邮件函数 
    def send_mail(to_list, sub, context): 
        '''''
        to_list: 发送给谁
        sub: 主题
        context: 内容
        send_mail("xxx@126.com","sub","context")
        '''
        me = mail_user + "<"+mail_user+"@"+mail_postfix+">"    msg = MIMEText(context)     msg['Subject'] = 'python email test'    msg['From'] = sub    msg['To'] = ";".join(to_list)     try:         send_smtp = smtplib.SMTP()         send_smtp.connect(mail_host)         send_smtp.login(mail_user, mail_pass)         send_smtp.sendmail(me, to_list, msg.as_string())         send_smtp.close()         print 'success'        return True    except (Exception, e):         print(str(e))         print 'false'  send_mail(mailto_list,"test mail","你好")

  

