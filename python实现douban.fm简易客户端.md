# python实现douban.fm简易客户端

一个月前心血来潮用python实现了一个简单的douban.fm客户端，计划是陆续将其完善成为Ubuntu下可替代web版本的douban.fm客户端。但后
来因为事多，被一直搁着，没有再继续完善。就在昨天，一位园友在评论中提到了登录的实现，虽然最近依然事多，但突然很想实现这个功能。正好，前几天因为一些需要，曾用
python实现过网站登录，约摸估计这douban.fm的登录不会差太多。

  

## 关于网站身份验证

http协议被设计为无连接协议，但现实中，很多网站需要对用户进行身份识别，cookie就是为此而诞生的。当我们用浏览器浏览网站时，浏览器会帮我们透明的处理c
ookie。而我们现在要第三方登录网站，这就必须对cookie的工作流程有一定的了解。

  

另外，很多网站为了防止程序自动登录而使用了验证码机制，验证码的介入会使登录过程变得麻烦，但也还不算太难处理。

  

## 实际中douban.fm的登录流程

为了模拟一个干净（不使用已有cookie）的登录流程，我使用chromium的隐身模式。

![](http://www.pythontab.com/uploadfile/2013/0115/20130115111012406.jpg)  

  

观察请求和响应头，可以看到，第一次请求的请求头是没有Cookie字段的，而服务器的响应头中包含着Set-
Cookie字段，这告诉浏览器下次请求该网站时需要携带Cookie。

  

这里我注意到了一个有意思的现象，访问douban.fm，实际中经过了3次重定向。当然，一般来说我们并不需要关注这些细节，浏览器和高级的httplib会透明的
处理重定向，但如果使用底层的C Socket，就必须小心的处理这些重定向。

  

点击登录按钮，浏览器发起几个新的请求，其中有几个至关重要的请求，这几个请求是我们第三方登录douban.fm的关键所在。

  

首先，有一条请求的URL是http://douban.fm/j/new_captcha，请求该URL，服务器会返回一个随机字符串，这有什么用呢？（其实是个验
证码）

再看下一条请求，http://douban.fm/misc/captcha?size=m&id=0iPlm837LsnSsJTMJrf5TZ7e，这条请求会
返回验证码。原来如此，请求http://douban.fm/j/new_captcha，将服务器返回的字符串作为下一条请求的id参数值。

  

我们可以写一段python代码来验证我们的想法。

  

值得注意的是**python提供了3个http库，httplib、urllib和urllib2**，能透明处理cookie的是urllib2，想我之前用ht
tplib手动处理cookie，那个痛苦啊。

代码如下：  

    
    
    opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(CookieJar()))
    captcha_id = opener.open(urllib2.Request('http://douban.fm/j/new_captcha')).read().strip('"') 
    captcha = opener.open(urllib2.Request('http://douban.fm/misc/captcha?size=m&id=' + captcha_id)).read())
    file = open('captcha.jpg', 'wb')
    file = write(captcha)
    file.close()

这段代码实现了验证码的下载。

接着，我们填写表单，并提交。

可以看到，登录表单的目标地址为http://douban.fm/j/login，参数有：

source: radio

alias: 用户名

form_password: 密码

captcha_solution: 验证码

captcha_id: 验证码ID

task: sync_channel_list

接下来要做的是用python构造一个表单。

    
    
    opener.open(
        urllib2.Request('http://douban.fm/j/login'),
        urllib.urlencode({
            'source': 'radio',
            'alias': username,
            'form_password': password,
            'captcha_solution': captcha,
            'captcha_id': captcha_id,
            'task': 'sync_channel_list'}))

服务器返回的数据格式是json，具体格式这里不赘诉了，大家可以自己测试。

  

我们怎么知道登录是否起作用了呢？是了，之前的文章提到过channel=-3为红心兆赫，是用户的收藏列表，没有登录是获取不到该频道的播放列表的。请求http:
//douban.fm/j/mine/playlist?type=n&channel=-3，如果返回你自己收藏过的音乐列表，那么就说明登录起作用了。

  

代码整理

结合之前的版本和新增的登录功能，再加上命令行参数处理、频道选择，一个稍稍完善的douban.fm就完成的。

    
    
    View Code 
     #!/usr/bin/python
     # coding: utf-8
      
     import sys
     import os
     import subprocess
     import getopt
     import time
     import json
     import urllib
     import urllib2
     import getpass
     import ConfigParser
     from cookielib import CookieJar
      
     # 保存到文件
     def save(filename, content):
         file = open(filename, 'wb')
         file.write(content)
         file.close()
      
      
     # 获取播放列表
     def getPlayList(channel='0', opener=None):
         url = 'http://douban.fm/j/mine/playlist?type=n&channel=' + channel
         if opener == None:
             return json.loads(urllib.urlopen(url).read())
         else:
             return json.loads(opener.open(urllib2.Request(url)).read())
      
      
     # 发送桌面通知
     def notifySend(picture, title, content):
         subprocess.call([
             'notify-send',
             '-i',
             os.getcwd() + '/' + picture,
             title,
             content])
      
      
     # 登录douban.fm
     def login(username, password):
         opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(CookieJar()))
         while True:
             print '正在获取验证码……'
             captcha_id = opener.open(urllib2.Request(
                 'http://douban.fm/j/new_captcha')).read().strip('"')
             save(
                 '验证码.jpg',
                 opener.open(urllib2.Request(
                     'http://douban.fm/misc/captcha?size=m&id=' + captcha_id
                 )).read())
             captcha = raw_input('验证码: ')
             print '正在登录……'
             response = json.loads(opener.open(
                 urllib2.Request('http://douban.fm/j/login'),
                 urllib.urlencode({
                     'source': 'radio',
                     'alias': username,
                     'form_password': password,
                     'captcha_solution': captcha,
                     'captcha_id': captcha_id,
                     'task': 'sync_channel_list'})).read())
             if 'err_msg' in response.keys():
                 print response['err_msg']
             else:
                 print '登录成功'
                 return opener
      
      
     # 播放douban.fm
     def play(channel='0', opener=None):
         while True:
             if opener == None:
                 playlist = getPlayList(channel)
             else:
                 playlist = getPlayList(channel, opener)
              
             if playlist['song'] == []:
                 print '获取播放列表失败'
                 break
                     picture,
      
             for song in playlist['song']:
                 picture = 'picture/' + song['picture'].split('/')[-1]
      
                 # 下载专辑封面
                 save(
                     picture,
                     urllib.urlopen(song['picture']).read())
      
                 # 发送桌面通知
                 notifySend(
                     picture,
                     song['title'],
                     song['artist'] + '\n' + song['albumtitle'])
      
                 # 播放
                 player = subprocess.Popen(['mplayer', song['url']])
                 time.sleep(song['length'])
                 player.kill()
      
      
     def main(argv):
         # 默认参数
         channel = '0'
         user = ''
         password = ''
      
         # 获取、解析命令行参数
         try:  
             opts, args = getopt.getopt(
                 argv, 'u:p:c:', ['user=', 'password=', 'channel='])  
         except getopt.GetoptError as error:
             print str(error)
             sys.exit(1)
      
         # 命令行参数处理
         for opt, arg in opts:
             if opt in ('-u', '--user='):
                 user = arg
             elif opt in ('-p', '--password='):
                 password = arg
             elif opt in ('-c', '--channel='):
                 channel = arg
      
         if user == '':
             play(channel)
         else:
             if password == '':
                 password = getpass.getpass('密码：')
             opener = login(user, password)
             play(channel, opener)
      
      
     if __name__ == '__main__':
         main(sys.argv[1:])

运行效果图![](http://www.pythontab.com/uploadfile/2013/0115/20130115111024352.png)  

