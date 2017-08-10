# PyPy 和 CPython 的性能比较测试

最近我在维基百科上完成了一些数据挖掘方面的任务。它由这些部分组成：

解析enwiki-pages-articles.xml的维基百科转储；

把类别和页存储到MongoDB里面；

对类别名称进行重新分门别类。

我对CPython 2.7.3和PyPy 2b的实际任务性能进行了测试。我使用的库是：

redis 2.7.2

pymongo 2.4.2

此外CPython是由以下库支持的：

hiredis

pymongo c-extensions

测试主要包含数据库解析，所以我没预料到会从PyPy得到多大好处（何况CPython的数据库驱动是C写的）。

下面我会描述一些有趣的结果。

  

抽取维基页名

  

我需要在所有维基百科的类别中建立维基页名到page.id的联接并存储重新分配好的它们。最简单的解决方案应该是导入enwiki-page.sql（定义了一个R
DB表）到MySQL里面，然后传输数据、进行重分配。但我不想增加MySQL需求（有骨气！XD）所以我用纯Python写了一个简单的SQL插入语句解析器
，然后直接从enwiki-page.sql导入数据，进行重分配。

这个任务对CPU依赖更大，所以我再次看好PyPy。

/ time

PyPy 169.00s 用户态 8.52s 系统态 90% CPU

CPython 1287.13s 用户态 8.10s 系统态 96% CPU

我也给page.id->类别做了类似的联接（我笔记本的内存太小了，不能保存供我测试的信息了）。

  

从enwiki.xml中筛选类别

  

为了方便工作，我需要从enwiki-pages-articles.xml中过滤类别，并将它们存储相同的XML格式的类别。因此我选用了SAX解析器，在PyPy
和CPython中都适用的包装器解析。对外的原生编译包（同事在PyPy和CPython 中） 。

代码非常简单：

    
    
    class WikiCategoryHandler(handler.ContentHandler):
        """Class which detecs category pages and stores them separately
        """
        ignored = set(('contributor', 'comment', 'meta'))
     
        def __init__(self, f_out):
            handler.ContentHandler.__init__(self)
            self.f_out = f_out
            self.curr_page = None
            self.curr_tag = ''
            self.curr_elem = Element('root', {})
            self.root = self.curr_elem
            self.stack = Stack()
            self.stack.push(self.curr_elem)
            self.skip = 0
     
        def startElement(self, name, attrs):
            if self.skip>0 or name in self.ignored:
                self.skip += 1
                return
            self.curr_tag = name
            elem = Element(name, attrs)
            if name == 'page':
                elem.ns = -1
                self.curr_page = elem
            else:   # we don't want to keep old pages in memory
                self.curr_elem.append(elem)
            self.stack.push(elem)
            self.curr_elem = elem
     
        def endElement(self, name):
            if self.skip>0:
                self.skip -= 1
                return
            if name == 'page':
                self.task()
                self.curr_page = None
            self.stack.pop()
            self.curr_elem = self.stack.top()
            self.curr_tag = self.curr_elem.tag
     
        def characters(self, content):
            if content.isspace(): return
            if self.skip == 0:
                self.curr_elem.append(TextElement(content))
                if self.curr_tag == 'ns':
                    self.curr_page.ns = int(content)
     
        def startDocument(self):
            self.f_out.write("<root>\n")
     
        def endDocument(self):
            self.f_out.write("<\root>\n")
            print("FINISH PROCESSING WIKIPEDIA")
     
        def task(self):
            if self.curr_page.ns == 14:
                self.f_out.write(self.curr_page.render())
     
     
    class Element(object):
        def __init__(self, tag, attrs):
            self.tag = tag
            self.attrs = attrs
            self.childrens = []
            self.append = self.childrens.append
     
        def __repr__(self):
            return "Element {}".format(self.tag)
     
        def render(self, margin=0):
            if not self.childrens:
                return u"{0}<{1}{2} />".format(
                    " "*margin,
                    self.tag,
                    "".join([' {}="{}"'.format(k,v) for k,v in {}.iteritems()]))
            if isinstance(self.childrens[0], TextElement) and len(self.childrens)==1:
                return u"{0}<{1}{2}>{3}</{1}>".format(
                    " "*margin,
                    self.tag,
                    "".join([u' {}="{}"'.format(k,v) for k,v in {}.iteritems()]),
                    self.childrens[0].render())
     
            return u"{0}<{1}{2}>\n{3}\n{0}</{1}>".format(
                " "*margin,
                self.tag,
                "".join([u' {}="{}"'.format(k,v) for k,v in {}.iteritems()]),
                "\n".join((c.render(margin+2) for c in self.childrens)))
     
    class TextElement(object):
        def __init__(self, content):
            self.content = content
     
        def __repr__(self):
            return "TextElement" def render(self, margin=0):
            return self.content

  

Element和TextElement元素包换tag和body信息，同时提供了一个方法来渲染它。

下面是我想要的PyPy和CPython比较结果。

/ time

PyPy 2169.90s

CPython 4494.69s

我很对PyPy的结果很吃惊。

计算有趣的类别集合

  

我曾经想要计算一个有趣的类别集合——在我的一个应用背景下，以Computing类别衍生的一些类别为开始进行计算。为此我需要构建一个提供类的类图——子类关系图
。

结构类——子类关系图

  

这个任务使用MongoDB作为数据来源，并对结构进行重新分配。算法是：

    
    
    for each category.id in redis_categories (it holds *category.id -> category title mapping*) do:
        title = redis_categories.get(category.id)
        parent_categories = mongodb get categories for title
        for each parent_cat in parent categories do:
            redis_tree.sadd(parent_cat, title) # add to parent_cat set title

  

抱歉写这样的伪码，但我想这样看起来更加紧凑些。

所以说这个任务仅把数据从一个数据库拷贝到另一个。这里的结果是MongoDB预热完毕后得出的（不预热的话数据会有偏差——这个Python任务只耗费约10%的C
PU）。计时如下：

/ time

PyPy 175.11s 用户态 66.11s 系统态 64% CPU

CPython 457.92s 用户态 72.86s 系统态 81% CPU

  

遍历redis_tree（再分配过的树）

  

如果我们有redis_tree数据库，仅剩的问题就是遍历Computing类别下所有可实现的结点了。为避免循环遍历，我们需要记录已访问过的结点。自从我想测试
Python的数据库性能，我就用再分配集合列来解决这个问题。

/ time

PyPy 14.79s 用户态 6.22s 系统态 69% CPU 30.322 总计

CPython 44.20s 用户态 13.86s 系统态 71% CPU 1:20.91 总计

说实话，这个任务也需要构建一些tabu list（禁止列表）——来避免进入不需要的类别。但那不是本文的重点。

  

结论

进行的测试仅仅是我最终工作的一个简介。它需要一个知识体系，一个我从抽取维基百科中适当的内容中得到的知识体系。

PyPy相比CPython，在我这个简单的数据库操作中，提高了2-3倍的性能。（我这里没有算上SQL解析器，大约8倍）

多亏了PyPy，我的工作更加愉悦了——我没有改写算法就使Python有了效率，而且PyPy没有像CPython一样把我的CPU弄挂了，以至于一段时间内我没法
正常的使用我的笔记本了（看看CPU时间占的百分比吧）。

任务几乎都是数据库操作，而CPython有一些加速的乱七八糟的C语言模块。PyPy不使用这些，但结果却更快！

我的全部工作需要大量的周期，所以我真高兴能用PyPy。

  

