# nltk 之 snowball 提取词干

机器学习中很重要的应用场景就是机器自动分类，而分类的关键是词干提取。所以我们要用到snowball。下面说一下snowball 提取词干的两种方法。

两种方法：

方法一：

    
    
    >>> from nltk import SnowballStemmer
    >>> SnowballStemmer.languages # See which languages are supported
    ('danish', 'dutch', 'english', 'finnish', 'french', 'german', 'hungarian',
     'italian', 'norwegian', 'porter', 'portuguese", 'romanian', 
     'russian', 'spanish', 'swedish')
    >>> stemmer = SnowballStemmer("german") # Choose a language
    >>> stemmer.stem(u"Autobahnen") # Stem a word
    u'autobahn'
    
    
    但是当你知道你使用的语言场景的时候可以使用下面的方法直接调用：
    
    
    方法二：
    
    
    >>> ps = nltk.stem.snowball.PortugueseStemmer()
    >>> ps.stem('celular')
    u'celul'
    >>> ps.stem('celular')
    u'celul'

  

  

