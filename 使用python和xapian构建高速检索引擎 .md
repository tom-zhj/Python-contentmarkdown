# 使用python和xapian构建高速检索引擎

首先弄明白几个概念：Documents 、terms and
posting在信息检索(IR)中，我们企图要获取的项称之“document”，每一个document是被一个terms集合所描述的。
“document”和“term”这两个词汇是IR中的术语，它们是来自“图书馆管理学”的。通常一个document认为是一块文本，（Usually a
document is thought of as a piece of text, most likely in a machine readable
form）,
而一个term则是一个词语或短语以用作描述document的，在document中大多数会存在着多个term，例如某个document是跟_口腔_ _卫生_
相关的，那么可能会存在着以下的terms：“tooth”、“teeth”、“toothbrush”、“decay”、“cavity”、“plaque”或“d
iet”等等。

如果在一个IR系统中，存在一个名为D的document，此document被一个名为t的term所描述，那么t被认为索引了D，可以用以下式子表
示：t->D。在实际应用的一个IR系统中通常是多个documents，如D1, D2, D3 …组成的集合，且有多个term，如t1, t2, t3
…组成的集合，从而有以下关系：ti -> Dj。

如果某个特定的term索引了某个特定的document，那么称之为posting，说白了posting就是带position信息的term，在相关度检索中可
能有一定的用途的。

给定一个名为D的document，存在着一个terms列表索引着它，我们称之为D的term list。

给定一个名为t的term，它索引着一个documents列表，这称之为t的posting list（使用“Document
list”可能会在叫法上更一致，但听起来过于空泛）。

在一个存在于计算机的IR系统中，terms是存储于索引文件中的。term可以用作有效地查找它的posting list，在posting
list里，每一个document带有一个很短的标识符，就是document id。简单来说，一个posting list可以被认为是一个由document
ids组成的集合，而term list则是一个字符串组成的集合。在某些IR系统的内部是使用数字来表示term的，因此在这些系统中，term
list则是数字组成的集合，而Xapian则不是这样，它使用原汁原味的term，而使用前缀来压缩存储空间。

Terms不一定是要是document中出现的词语，通常它们会被转换为小写，而且往往它们被词干提取算法处理过，因此通过一个值为“connect”
的term可能会检索出一系列的词语，例“connect”、“connects”、“connection”或“connected”等，而一个词语
也可能产生多个的terms，例如你会将提取出的词干和未提取的词语都索引起来。当然，这可能只适用于英语、法语或拉丁语等欧美系列的语言，而中文的分词
则有很大的区别，总的来说，欧美语系的语言分词与中文分词有以下的区别：

l. 拿英语来说，通常情况下英语的每一个词语之间是用空格来隔开的，而中文则不然，甚至可以极端到整篇文章都不出现空格或标点符号。 2.
像上面提到的，“connect”、“connects”、“connection”或“connected”分别的意思“动词性质的连接”、“动词性质
的第三人称的连接”、“名称性质的连接”或“连接的过去式”，但在中文里，用“连接”就可以表示全部了，几乎不需要词干提取。这意味着英语的各种词性大部
分是有章可循的，而中文的词性则是天马行空的。 3.
第二点只是中文分词非常困难的一个缩影，要完全正确地标识出某个句子的语意是很困难的，例如“中华人民共和国成立了”这个句子，可以分出“中华”、“华
人”、“人民”、“共和国”、“成立”等词语，不过其中“华人”跟这个句子其实关系不大。咋一眼看上去很简单，但机器那有这么容易懂这其中的奥妙呢？

Values

Values是附加在document上一种元数据，每一个document可以有多个values，这些values通过不同的数字来标识。
Values被设计成在匹配过程中快速地访问，它们可以用作排序、排队多余重复的document和范围检索等用途。虽然values并没有长度限制，但
最好让它们尽可能短，如果你仅仅是想存储某个字段以便作为结果显示，那么建议您最好将它们保存在document的data中。

Document data

每一个Document只有一个data，可以是任意类型格式的数据，当然在存储的时候请先转换为字符串。这听上去可能有点古怪，实情是这样的：如果要存
储的数据是文本格式，则可以直接存储；如果要存储的数据是各种的对象，请先序列化成二进制流再保存，而在读取的时候反序列化读取。

posting

posting是带position的term.

    
    
    
    # -*- coding: gb18030 -*-
    import xapian
    testdatas = [u'abc test python1',u'abcd testing python2']
    def buildtest():
        database = xapian.WritableDatabase('indexes/', xapian.DB_CREATE_OR_OPEN)
        stemmer = xapian.Stem("english")
        for data in testdatas:
            doc = xapian.Document()
            doc.set_data(data)
            for term in data.split():
                doc.add_term(term)
            database.add_document(doc)
    if __name__ == '__main__':
        buildtest()

执行后,当前目录下生成索引库。  

[sh]

[ec2-user@ip-10-167-6-221 indexes]$ ll

总用量 52

-rw-rw-r-- 1 ec2-user ec2-user    0  7月 28 16:06 flintlock

-rw-rw-r-- 1 ec2-user ec2-user   28  7月 28 16:06 iamchert

-rw-rw-r-- 1 ec2-user ec2-user   13  7月 28 16:06 postlist.baseA

-rw-rw-r-- 1 ec2-user ec2-user   14  7月 28 16:06 postlist.baseB

-rw-rw-r-- 1 ec2-user ec2-user 8192  7月 28 16:06 postlist.DB

-rw-rw-r-- 1 ec2-user ec2-user   13  7月 28 16:06 record.baseA

-rw-rw-r-- 1 ec2-user ec2-user   14  7月 28 16:06 record.baseB

-rw-rw-r-- 1 ec2-user ec2-user 8192  7月 28 16:06 record.DB

-rw-rw-r-- 1 ec2-user ec2-user   13  7月 28 16:06 termlist.baseA

-rw-rw-r-- 1 ec2-user ec2-user   14  7月 28 16:06 termlist.baseB

-rw-rw-r-- 1 ec2-user ec2-user 8192  7月 28 16:06 termlist.DB

我们下篇再介绍如何去查询索引。

  

