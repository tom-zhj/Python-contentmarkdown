# 使用python进行汉语分词

目前我常常使用的分词有结巴分词、NLPIR分词等等

最近是在使用结巴分词，稍微做一下推荐，还是蛮好用的。

一、结巴分词简介

利用结巴分词进行中文分词，基本实现原理有三：

基于Trie树结构实现高效的词图扫描，生成句子中汉字所有可能成词情况所构成的有向无环图（DAG)

采用了动态规划查找最大概率路径, 找出基于词频的最大切分组合

对于未登录词，采用了基于汉字成词能力的HMM模型，使用了Viterbi算法

二、安装及使用（Linux）

1.下载工具包，解压后进入目录下，运行：python setup.py install

Hint：a.一个良好的习惯是，对于下载下来的软件，先看readme ，再进行操作。（没有阅读readme，直接尝试+百度，会走很多弯路）；

　　   b.当时运行安装命令时，出现错误：no permission！  （有些人可能会遇见这种问题，这是因为权限不够的。 执行：sudo !!
其中“!!”表示上一条命令，这里指的就是上面的安装命令），使用sudo后便可正常运行。

  

2.在使用结巴做分词的时候，一定会用的函数是：jieba.cut（arg1，arg2）；这是用于分词的函数，我们只需要了解以下三点，便可使用

　　a.cut方法接受两个输入参数：第一个参数（arg1）为需要进行分词的字符串，arg2参数用来控制分词模式。

　　分词模式分为两类：默认模式，试图将句子最精确地切开，适合文本分析;全模式，把句子中所有的可以成词的词语都扫描出来，适合搜索引擎

 　　b.待分词的字符串可以是gbk字符串、utf-8字符串或者unicode

使用Python的人要注意编码问题，Python是基于ASCII码来处理字符的，当出现不属于ASCII的字符时（比如在代码中使用汉字），会出现错误信息：“A
SCII codec can't encode character”，解决方案是在文件顶部加入语句： #! -*- coding:utf-8 -*-
来告诉Python的编译器：“我这个文件是用utf-8进行编码的，你要进行解码的时候，请用utf-8”。（这里记住，这个命令一定要加在文件的最顶部，如果不在
最顶部，编码问题就依然存在，得不到解决）关于编码的转换，可以参考博文（ps：个人理解“import sys    reload(sys)
sys.setdefaultencoding('utf-8')”这几句话与“#! -*- coding:utf-8 -*- ”等价）

 　  c.jieba.cut返回的结构是一个可迭代的generator，可以使用for循环来获得分词后得到的每一个词语(unicode)，也可以用list
(jieba.cut(...))转化为list

3.以下举例jieba中提供的一个使用方法作为说明：

    
    
    #! -*- coding:utf-8 -*-
    import jieba
    seg_list = jieba.cut("我来到北京清华大学", cut_all = True)
    print "Full Mode:", ' '.join(seg_list)
     
    seg_list = jieba.cut("我来到北京清华大学")
    print "Default Mode:", ' '.join(seg_list)

输出结果为：

    
    
    Full Mode: 我/ 来/ 来到/ 到/ 北/ 北京/ 京/ 清/ 清华/ 清华大学/ 华/ 华大/ 大/ 大学/ 学  
    Default Mode: 我/ 来到/ 北京/ 清华大学

三、结巴中文分词的其他功能

1、添加或管理自定义词典

结巴的所有字典内容存放在dict.txt，你可以不断的完善dict.txt中的内容。

2、关键词抽取

通过计算分词后的关键词的TF/IDF权重，来抽取重点关键词。

  

