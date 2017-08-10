# 机器学习之nltk download出错：Error connecting to server: [Errno -2]

机器学习常用到python的自然语言处理框架NLTK，这个是机器学习的常用包，在使用过程中会遇到不少问题。我会和大家分享在这其中的一些经验。

今天闲来说一下安装，在安装中出现的download错误。

  

>>> import nltk

>>> nltk.download()

NLTK Downloader

\---------------------------------------------------------------------------

   d) Download      l) List      c) Config      h) Help      q) Quit

\---------------------------------------------------------------------------

Downloader> l

Packages:

Error connecting to server: [Errno -2] Name or service not known

经过推测，是服务器无法连接下载服务器地址导致的。

查看一下nltk download配置

Downloader> c

Data Server:

 - URL: <http://nltk.googlecode.com/svn/trunk/nltk_data/index.xml>

 - 3 Package Collections Available

 - 74 Individual Packages Available

Local Machine:

 - Data directory: /home/wym/nltk_data

然后

> curl http://nltk.googlecode.com/svn/trunk/nltk_data/index.xml

报错：curl: (6) Couldn't resolve host 'nltk.googlecode.com'

google的地址肯定是被墙掉了（唉，大家懂的）

解决办法：

修改dns地址，

> vim /etc/resolv.conf

修改nameserver为：  nameserver  8.8.8.8

问题解决

  

