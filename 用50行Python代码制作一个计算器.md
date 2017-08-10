# 用50行Python代码制作一个计算器

简介

在这篇文章中，我将向大家演示怎样向一个通用计算器一样解析并计算一个四则运算表达式。当我们结束的时候，我们将得到一个可以处理诸如
1+2*-(-3+2)/5.6+3样式的表达式的计算器了。当然，你也可以将它拓展的更为强大。

我本意是想提供一个简单有趣的课程来讲解 语法分析 和 正规语法（编译原理内容）。同时，介绍一下 PlyPlus,这是一个我断断续续改进了好几年的语法解析
接口。作为这个课程的附加产物，我们最后会得到完全可替代eval（）的一个安全的四则运算器。

如果你想在自家的电脑上试试本文中给的例子的话，你应该先安装 PlyPlus ,使用命令pip install plyplus
。（译者注：pip是一个包管理系统，用来安装用python写的软件包，具体使用方法大家可以百度之或是google之，就不赘述了。）

本篇文章需要对python的继承使用有所了解。

语法

  

对于那些不懂的如何解析和正式语法工作的人而言，这里有一个快速的概览：正式语法是用来解析文本的一些不同层面的规则。每一个规则都描述了相对应的那部分输入的文本是
如何组成的。

这里是一个用来展示如何解析1+2+3+4的例子：

  

Rule #1 - add  IS MADE OF  add + number

                      OR  number + number

或者用 EBNF:

add: add'+'number

  | number'+'number

  ;

解析器每次都会寻找add+number或者number+number，找到一个之后就会将其转换成add。基本上而言，每一个解析器的目标都在于尽可能的找到最高
层次的表达式抽象。

以下是解析器的每个步骤：

number + number + number + number

第一次转换将所有的Number变成“number”规则

[number + number] + number + number

解析器找到了它的第一个匹配模式！

[add + number] + number

在转换成一个模式之后，它开始寻找下一个

[add + number]

add

这些有次序的符号变成了一个层次上的两个简单规则:
number+number和add+number。这样，只需要告诉计算机如果解决这两个问题，它就能解析整个表达式。事实上，无论多长的加法序列，它都能解决!
这就是形式文法的力量。

运算符优先级

  

算数表达式并不仅仅是符号的线性增长，运算符创造了一个隐式的层次结构，这非常适合用形式文法来表示:

1 + 2 * 3 / 4 - 5 + 6

这相当于:

1 + (2 * 3 / 4) - 5 + 6

我们可以通过嵌套规则表示此语法中的结构:

add: add+mul

  | mul'+'mul

  ;

mul: mul '*; number

  | number'*'number

  ;

通过将add设为操作mul而不是number，我们就得到了乘法优先的规则。

让我们在脑海中模拟一下使用这个神奇的解析器来分析1+2*3*4的过程：

number + number * number * number

number + [number * number] * number

解析器不知道number+number的结果，所以这是它（解析器）的另一个选择

number + [mul * number]

number + mul

现在我们遇到了一点困难!
解析器不知道如何处理number+mul。我们可以区分这种情况，但是如果我们继续探索下去，就会发现有很多不同的没有考虑到得可能，比如mul+number,
add+number, add+add, 等等。

那么我们应该怎么做呢？

幸运的是，我们可以做一点小“把戏”：我们可以认为一个number本身是一个乘积，并且一个乘积本身是一个和！

这种思路一开始看起来有点古怪，不过它的确是有意义的：

add: add'+'mul

  | mul'+'mul

  | mul

  ;

mul: mul'*'number

  | number'*'number

  | number

  ;

但是如果 mul能够变成 add, 且 number能够变成 mul ， 有些行的内容就变得多余了。丢弃它们，我们就得到了：

add: add'+'mul

  | mul

  ;

mul: mul'*'number

  | number

  ;

让我们来使用这种新的语法来模拟运行一下1+2*3*4：

number + number * number * number

现在没有一个规则是对应number*number的了，但是解析器可以“变得有创造性”

number + [number] * number * number

number + [mul * number] * number

number + [mul * number]

[number] + mul

[mul] + mul

[add + mul]

add

成功了！！！

如果你觉得这个很奇妙，那么尝试着去用另一种算数表达式来模拟运行一下，然后看看表达式是如何用正确的方式来一步步解决问题的。或者等着阅读下一节中的内容，看看计算
机是如何一步步运行出来的！

运行解析器

现在我们对于如何让我们的语法运作起来已经有了非常不错的想法了，那就写一个实际的语法来应用一下吧:

start: add;            // 这是最高层

add: add add_symbol mul | mul;

mul: mul mul_symbol number | number;

number:'[d.]+';      // 十进制数的正则表达式

mul_symbol:'*'|'/';// Match * or /

add_symbol:'+'|'-';// Match + or -

你可能想要复习一下正则表达式，但不管怎样，这个语法都非常直截了当。让我们用一个表达式来测试一下吧:

>>>fromplyplusimportGrammar

>>> g=Grammar(&quot;&quot;&quot;...&quot;&quot;&quot;)

>>>printg.parse('1+2*3-5').pretty()

start

 add

   add

     add

       mul

         number

           1

     add_symbol

       +

     mul

       mul

         number

           2

       mul_symbol

         *

       number

         3

   add_symbol

     -

   mul

     number

       5

干得漂亮!

仔细研究一下这棵树，看看解析器选择了什么层次。

如果你希望亲自运行这个解析器，并使用你自己的表达式，你只需有Python即可。安装Pip和PlyPlus之后，将上面的命令粘贴到Python内(记得将'..
.'替换为实际的语法哦~)。

使树成形

Plyplus会自动创建一棵树，但它并不一定是最优的。将number放入到mul和将mul放入到add非常有利于创建一个阶层，现在我们已经有了一个阶层那它们
反而会成为一个负担。我们告诉Plyplus对它们加前缀去“展开”（i.e.删除）规则。

碰到一个@常常会展开一个规则，一个#则会压平它，一个?会在它有一个子结点时展开。在这种情况下，?就是我们所需要的。

start: add;

?add: add add_symbol mul | mul;      // Expand add if it's just a mul

?mul: mul mul_symbol number | number;// Expand mul if it's just a number

number:'[d.]+';

mul_symbol:'*'|'/';

add_symbol:'+'|'-';

在新语法下树是这样的：

>>> g=Grammar(&quot;&quot;&quot;...&quot;&quot;&quot;)

>>>printg.parse('1+2*3-5').pretty()

start

 add

   add

     number

       1

     add_symbol

       +

     mul

       number

         2

       mul_symbol

         *

       number

         3

   add_symbol

     -

   number

     5

哦，这样变得简洁多了，我敢说，它是非常好的。

括号的处理及其它特性

  

目前为止，我们还明显缺少一些必须的特性：括号，单元运算符(-(1+2))，及表达式中间允许存在空字符。其实这些特性都很容易就能实现，下面我们来尝试一下。

需要先引入一个重要的概念：原子。在一个原子里面（括号中及单元运算）发生的所有操作都优先于所有加法或乘法运算（包括位操作）。由于原子只是一个优先级的构造器，并
无语法意义，帮我们加上"@"符号以确保在编译时它被能展开。

允许空格出现在表达式内最简单的方法就是使用这种解释方式：add SPACE add_symbol SPACE mul | mul;
但个解释结果啰嗦且可读性差。所有，我们需要令Plyplus总是忽略空格。

下面是完整的语法，包容了以上所述特性：

start: add;

?add: (add add_symbol)? mul;

?mul: (mul mul_symbol)? atom;

@atom: neg | number |'('add')';

neg:'-'atom;

number:'[d.]+';

mul_symbol:'*'|'/';

add_symbol:'+'|'-';

WHITESPACE:'[ t]+'(%ignore);

请确保理解这个语法再进入下一步：计算！

运算

  

现在，我们已经可以将一个表达式转化成一棵分层树了，只需要逐分支地扫描这棵树，便可得到最终结果。

我们现在要开始编写代码了，在此之前，我需要对这棵树做两点解释：

   1.每个分支都是包含如下两个属性的实例：

       头(head)：规则的名字(例如add或者number);

       尾(tail)：包含所有与其匹配的子规则的列表。

   2.Plyplus默认会删除不必要的标记。在本例中，'( ' ，')' 和 '-' 会被删除。但add和mul会有自己的规则，Plyplus会知道它们
是必须的，从而不会被删除它们。如果你需要保留这些标记，可以手动关掉这项功能，但从我的经验来看，最好不要这样做，而是手动修改相关语法效果更佳。

言归正传，现在我们开始编写代码。我们将用一个非常简单的转换器来扫描这棵树。它会从最外面的分支开始扫描，直到到达根节点为止，而我们的工作是告诉它如何扫描。如果
一切顺利的话，它将总会从最外层开始扫描!让我们看看具体的实现吧。

>>>importoperator as op

>>>fromplyplusimportSTransformer

classCalc(STransformer):

   def_bin_operator(self, exp):

       arg1, operator_symbol, arg2=exp.tail

       operator_func={'+': op.add,

                         '-': op.sub,

                         '*': op.mul,

                         '/': op.div }[operator_symbol]

       returnoperator_func(arg1, arg2)

   number     =lambdaself, exp:float(exp.tail[0])

   neg        =lambdaself, exp:-exp.tail[0]

   __default__=lambdaself, exp: exp.tail[0]

   add=_bin_operator

   mul=_bin_operator

每个方法都对应一个规则。如果方法不存在的话，将调用__default__方法。我们在其中省略了start,add_symbol和mul_symbol，因为它
们只会返回自己的分支。

我使用了float()来解析数字，这是个懒方法，但我也可以用解析器来实现。

为了使语句整洁，我使用了运算符模块。例如add基本上是 'lambda x,y: x+y'之类的。

OK，现在我们运行这段代码来检查一下结果。

>>> Calc().transform( g.parse('1 + 2 * -(-3+2) / 5.6 + 30'))

31.357142857142858

那么eval()呢？7

>>>eval('1 + 2 * -(-3+2) / 5.6 + 30')

31.357142857142858

成功了:)

最后一步：REPL

  

为了美观，我们把它封装到一个不错的计算器 REPL：

defmain():

   calc=Calc()

   whileTrue:

       try:

           s=raw_input('> ')

       exceptEOFError:

           break

       ifs=='':

           break

       tree=calc_grammar.parse(s)

       printcalc.transform(tree)

完整的代码可从这里获取：

https://github.com/erezsh/plyplus/blob/master/examples/calc.py

  

