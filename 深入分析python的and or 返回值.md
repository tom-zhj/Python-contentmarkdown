# 深入分析python的and or 返回值

在Python 中，and 和 or 执行布尔逻辑演算，如你所期待的一样，但是它们并不返回布尔值；而是，返回它们实际进行比较的值之一。

  

一、and:

    
    
    >>> 'a' and 'b'
    'b'
    >>> '' and 'b'
    ''
    >>> 'a' and 'b' and 'c'
    'c'

  

在布尔上下文中从左到右演算表达式的值，如果布尔上下文中的所有值都为真，那么 and 返回最后一个值。

如果布尔上下文中的某个值为假，则 and 返回第一个假值

  

二、or:

    
    
    >>> 'a' or 'b'
    'a'
    >>> '' or 'b'
    'b'
    >>> '' or [] or {}
    {}
    >>> 0 or 'a' or 'c'
    'a'

  

使用 or 时，在布尔上下文中从左到右演算值，就像 and 一样。如果有一个值为真，or 立刻返回该值

如果所有的值都为假，or 返回最后一个假值

注意 or 在布尔上下文中会一直进行表达式演算直到找到第一个真值，然后就会忽略剩余的比较值

  

三、and-or:

  

and-or 结合了前面的两种语法，推理即可。

    
    
    >>> a='first'
    >>> b='second'
    >>> 1 and a or b
    'first'
    >>> (1 and a) or b
    'first'
    >>> 0 and a or b
    'second'
    >>> (0 and a) or b
    'second'
    >>>

  

这个语法看起来类似于 C 语言中的 bool ? a : b 表达式。整个表达式从左到右进行演算，所以先进行 and 表达式的演算。 1 and
'first' 演算值为 'first'，然后 'first' or 'second' 的演算值为 'first'。

0 and 'first' 演算值为 False，然后 0 or 'second' 演算值为 'second'。

and-or主要是用来模仿 三目运算符 bool?a:b的，即当表达式bool为真，则取a否则取b。

and-or 技巧，bool and a or b 表达式，当 a 在布尔上下文中的值为假时，不会像 C 语言表达式 bool ? a : b 那样工作。

  

四、安全使用and-or

    
    
    
    >>> a=""
    >>> b="second"
    >>> (1 and [a] or [b])
    ['']
    >>> (1 and [a] or [b])[0]
    ''
    >>>

  

由于 [a] 是一个非空列表，所以它决不会为假。即使 a 是 0 或者 '' 或者其它假值，列表 [a] 也为真，因为它有一个元素。

一个负责的程序员应该将 and-or 技巧封装成一个函数：

    
    
    def choose(bool,a,b):
        return (bool and [a] or [b])[0]
    print  choose(1,'','second')    #''

  

