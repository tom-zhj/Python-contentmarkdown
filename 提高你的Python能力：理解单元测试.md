# 提高你的Python能力：理解单元测试

对于程序开发新手来说，一个最常见的困惑是测试的主题。他们隐约觉得“单元测试”是很好的，而且他们也应该做单元测试。但他们却不懂这个词的真正含义。如果这听起来像
是在说你，不要怕！在这篇文章中，我将介绍什么是单元测试，为什么它有用，以及如何对Python的代码进行单元测试。

### 什么是测试？

在讨论为什么测试很有用、怎样进行测试之前，让我们先花几分钟来定义一下“单元测试”究竟是什么。在一般的编程术语中，“测试”指的是通过编写可以调用的代码（独立于
你实际应用程序的代码）来帮助你确定程序中是否有错误。这并不能证明你的代码是正确的（在非常有限的情况下这是唯一的可能）。它只是报告了测试者认为的那种情况是否被
正确处理了。

注：当我使用“测试”一次时，我指的是“自动化测试”，即这些测试是在机器上运行的。“手动测试”则是一个人运行程序，并与它进行交互，从而发现漏洞，这是个独立的概
念。

测试可以检查出什么样的情况呢？语法错误是语言的意外误用，如

    
    
    my_list..append(foo)

后面多余的一个 “.“。逻辑错误是当算法（可以看成是“解决问题的方式”）不正确时引发的。可能程序员忘记Python是“零索引“的并且试图通过写

    
    
    print(my_string[len(my_string)])

（这样会引起IndexError）来打印出一个字符串中的最后一个字符。更大、更系统的错误也可以被检查出来。比如当用户输入一个大于100的数字、或者在网站检索
不可用的时候挂起此网站的话，程序会一直崩溃。

这些所有的错误都可以通过对代码的仔细测试检查出来。Unit testing,特指在一个分隔的代码单元中的测试。一个单元可以是整个模块，一个单独的类或者函数，
或者这两者间的任何代码。然而，重要的是，测试代码要与我们没有测试到的其他代码相互隔离（因为其它代码本身有错误的话会因此混淆测试结果）。考虑如下例子：

    
    
    def is_prime(number):
        """Return True if *number* is prime."""
        for element in range(number):
            if number % element == 0:
                return False
        return True
    def print_next_prime(number):
        """Print the closest prime number larger than *number*."""
        index = number
        while True:
            index += 1
            if is_prime(index):
                print(index)

你有两个函数，is_prime 和
print_next_prime。如果你想测试print_next_prime，我们就需要确定is_prime是正确的，因为print_next_prime
中调用了这个函数。在这种情况下，print_next_prime 函数是一个单元，is_prime
函数是另一个单元。由于单元测试每次只测试一个单元，因此我们需要仔细考虑怎样才能准确的测试 print_next_prime
?（更多的是关于之后怎样实现这些测试）。

因此，测试代码应该长什么样呢？如果上一个例子存在一个叫primes.py 的文件中，我们可以把测试代码写在一个叫 test_primes.py
的文件中。下面是 test_primes.py 中的最基本内容，比如下面这个测试样例：

    
    
    import unittest
    from primes import is_prime
    class PrimesTestCase(unittest.TestCase):
        """Tests for `primes.py`."""
        def test_is_five_prime(self):
            """Is five successfully determined to be prime?"""
            self.assertTrue(is_prime(5))
    if __name__ == '__main__':
        unittest.main()

这个文件通过一个test case:?test_is_five_prime. 创建了一个单元测试。通过Python内嵌的一个测试框架 unittest。当u
nittest.main()被调用时，任何一个以test开头命名的成员函数将被运行，他们是unittest.TestCase的一个派生类，并且是断言检查的。
如果我们通过输入python test_primes.py来运行测试，我们能够看到unittest框架在控制台上 的输出：

    
    
    $ python test_primes.py
    E
    ======================================================================
    ERROR: test_is_five_prime (__main__.PrimesTestCase)
    ----------------------------------------------------------------------
    Traceback (most recent call last):
    File "test_primes.py", line 8, in test_is_five_prime
        self.assertTrue(is_prime(5))
    File "/home/jknupp/code/github_code/blug_private/primes.py", line 4, in is_prime
        if number % element == 0:
    ZeroDivisionError: integer division or modulo by zero
    ----------------------------------------------------------------------
    Ran 1 test in 0.000s

单独的“E”表示的是我们单元测试的结果（如果它成功了，会打印出一个“.”）。我们可以看到我们的测试失败了，以及导致失败的那行代码，还有任何引发的异常信息。

### 为什么要测试？

在我们继续那个例子之前，要问个很重要的问题：“为什么测试对我来说有价值”？这是个公平的问题，也是那些对于代码测试不熟悉的人常问的问题。毕竟，测试需要一定的时
间，而我们完全可以用这些时间去编代码，为什么要测试而不是去做那些最有生产效率的事？

有很多答案可以有效的回答这个问题，我列出了以下几点：

测试可以保证你的代码在一系列给定条件下正常工作

测试确保了一系列条件下的正确性。语法错误基本上一定通过测试被查出来，一个代码单元的基本的逻辑也可以通过测试被检测出来，以确保一定条件下的正确性。再次，它不是
要证明代码是在任何条件下都正确的。我们只是简单的瞄准了一套比较完整的可能的条件（例如，你可以写一个测试来监测当你调用my_addition_function
(3, 'refrigerator), 的时候，但你不必为每个参数检测所有可能的字符串）

测试允许人们确保对代码的改动不会破坏现有的功能

重构代码时，这一点特别有用。如果没有测试到位，你就没法保证你的代码的改变没有破坏之前工作正常的东西。如果你希望更改或重写你的代码，并希望不会破坏任何东西，适
当的单元测试是很必要的。

测试迫使人们在不寻常条件的情况下思考代码，这可能会揭示出逻辑错误

编写测试强迫你去思考在非正常条件下你的代码可能遇到的问题。在上面的例子中，my_addition_function函数可以将两个数字相加。测试基本正确性的简
单测试将调用my_addition_function（2,2），并断言说结果是4。然而，进一步的测试可能会通过调用my_addition_function（
2.0,2.0）来测试该功能是否能正确进行浮点数的运算。防御性的编码原则表明你的代码应该能够在非法输入的情况下正常失效，因此测试时，当字符串类型被作为参数传
递到函数中时应当抛出一个异常。

良好的测试要求模块化，解耦代码，这是一个良好的系统设计的标志

单元测试的整体做法是通过代码的松散耦合使其变得更容易。如果你的应用程序代码直接调用数据库，例如，测试你应用程序的逻辑依赖于一个有效的数据库连接，并且测试数据
要存在于数据库中。另一方面，隔离了外部资源的代码在测试过程中更容易被模拟对象所替代。出于必要，（人们）设计的有测试能力的应用程序最终采用了模块化和松散耦合。

### 单元测试的剖析

通过继续之前的例子，我们将看到如何编写并组织单元测试。回想一下，primes.py包含以下代码：

    
    
    def is_prime(number):
        """Return True if *number* is prime."""
        for element in range(number):
            if number % element == 0:
                return False
        return True
    def print_next_prime(number):
        """Print the closest prime number larger than *number*."""
        index = number
        while True:
            index += 1
            if is_prime(index):
                print(index)

同时，文件test_primes.py包含如下代码：

    
    
    import unittest
    from primes import is_prime
    class PrimesTestCase(unittest.TestCase):
        """Tests for `primes.py`."""
        def test_is_five_prime(self):
            """Is five successfully determined to be prime?"""
            self.assertTrue(is_prime(5))
    if __name__ == '__main__':
        unittest.main()

### 做出断言

unittest是Python标准库中的一部分，并且也是我们开始“单元测试之旅”的一个好的起点。一个单元测试中包括一个或多个断言（一些声明被测试代码的一些属
性为真的语句）。会想你上学的时候“断言”这个词的字面意思就是“陈述事实”。在单元测试中，断言也是同样的作用。

self.assertTrue 更像是自我解释。它能声明传递过去的参数的计算结果为真。unittest.TestCase类包含了许多断言方法，所以一定要检查
列表并选择合适的方法进行测试。如果在每个测试中都用到assertTrue的话，则应该考虑一个反模式，因为它增加了测试中读者的认知负担。正确使用断言的方法应当
是使测试能够明确说明究竟是什么在被断言（例如，很明显?，只需扫一眼assertIsInstance 的方法名，就知道它要说明的是其参数）。

每个测试应该测试一个单独、有具体特性的代码，并且应该被赋予相关的命名。就单元测试发现机制的研究表明（主要在Python2.7+和3.2+版本中），测试方法应
该以test_为前缀命名。（这是可配置的，但是其目的是鉴别测试方法和非测试的实用方法）。如果我们把test_is_five_prime
的命名改为is_five_prime的话，运行python中的test_primes.py时会输出如下信息:

    
    
    $ python test_primes.py
    ----------------------------------------------------------------------
    Ran 0 tests in 0.000s
    OK

不要被上面信息中的“OK”所糊弄了，只有当什么测试都没真正运行的时候才会显示出“OK”！我认为一个测试也没跑其实应该显示个报错的，但是个人感觉放在一边，这是
一个你应该注意是行为，尤其是当通过程序运行来检查测试结果的时候（例如，一个持续的集成工具，像TracisCI）。

异常

让我们回到test_primes.py的实际内容中去，回忆一下运行python test_primes.py指令后的输出结果：

    
    
    $ python test_primes.py
    E
    ======================================================================
    ERROR: test_is_five_prime (__main__.PrimesTestCase)
    ----------------------------------------------------------------------
    Traceback (most recent call last):
    File "test_primes.py", line 8, in test_is_five_prime
        self.assertTrue(is_prime(5))
    File "/home/jknupp/code/github_code/blug_private/primes.py", line 4, in is_prime
        if number % element == 0:
    ZeroDivisionError: integer division or modulo by zero
    ----------------------------------------------------------------------
    Ran 1 test in 0.000s

这些输出告诉我们，我们一个测试的结果失败并不是因为一个断言失败了，而是因为出现了一个未捕获的异常。事实上，由于抛出了一个异常，unittest框架并没有能够
运行我们的测试就返回了。

这里的问题很明确：我们使用的求模运算的计算范围中包括了0，因此执行了一个除以0的操作。为了解决这个问题，我们可以很简单的将起始值由0变为2，并指出对0求模是
错误的，而对1求模则一直是真（并且一个素数只能被自身和1整除，因此我们无需检查1）。

### 解决问题

一次失败的测试使我们修改了代码。一旦我们改好了这个错误（将s_prime中的一行改为for element in range(2,
number):），我们就得到了如下输出：

    
    
    $ python test_primes.py
    .
    ----------------------------------------------------------------------
    Ran 1 test in 0.000s

现在错误已经改了，这是不是意味着我们应该删掉test_is_five_prime这个测试方法（因为很明显，它将不会一直能通过测试）？不应该删。由于通过测试是
最终目标的话单元测试应该尽量少的被删除。我们已经测试过is_prime的语法是有效的，并且，至少在一种情况下，它返回正确的结果。我们的目标是要建立一套能全部
通过的（单元测试的逻辑分组）测试，虽然有些一开始可能会失败。

test_is_five_prime用于处理一个“非特殊”的素数。让我们确保它也能正确处理非素数。将以下方法添加到PrimesTestCase类：

    
    
    def test_is_four_non_prime(self):
        """Is four correctly determined not to be prime?"""
        self.assertFalse(is_prime(4), msg='Four is not prime!')

请注意，这时我们给assert调用添加了可选的msg参数。如果该测试失败了，我们的信息将被打印到控制台，并给运行测试的人提供额外的信息。

### 边界情况

我们已经成功的测试了两种普通情况。现在让我们考虑边界情况下、或者那些不寻常或意外的输入的用例。当测试一个其范围是正整数的函数时，边界情况下的实例包括0、1、
负数和一个很大的数字。现在让我们来测试其中的一些。

添加一个对0的测试很简单。我们预计?is_prime(0)返回的是false，因为，根据定义，素数必须大于1。

    
    
    def test_is_zero_not_prime(self):
        """Is zero correctly determined not to be prime?"""
        self.assertFalse(is_prime(0))

可惜呀，输出是：

    
    
    python test_primes.py
    ..F
    ======================================================================
    FAIL: test_is_zero_not_prime (__main__.PrimesTestCase)
    Is zero correctly determined not to be prime?
    ----------------------------------------------------------------------
    Traceback (most recent call last):
    File "test_primes.py", line 17, in test_is_zero_not_prime
        self.assertFalse(is_prime(0))
    AssertionError: True is not false
    ----------------------------------------------------------------------
    Ran 3 tests in 0.000s
    FAILED (failures=1)

0被错误的判定为素数。我们忘记了，我们决定在数字范围中跳过0和1。让我们增加一个对他们的特殊检查。

    
    
    def is_prime(number):
        """Return True if *number* is prime."""
        if number in (0, 1):
            return False
        for element in range(2, number):
            if number % element == 0:
                return False
        return True

现在测试通过了。我们的函数应该怎样处理一个负数？在写这个测试用例之前就知道输出结果是很重要的。在这种情况下，任何负数都应该返回false。

    
    
    def test_negative_number(self):
        """Is a negative number correctly determined not to be prime?"""
        for index in range(-1, -10, -1):
            self.assertFalse(is_prime(index))

这里我们觉得检查从-1到-
9的所有数字。在一个循环中调用test方法是非常合法的，在一个测试中多次调用断言方法也可以。我们可以在下面用（更详细）的方式改写代码。

    
    
    def test_negative_number(self):
        """Is a negative number correctly determined not to be prime?"""
        self.assertFalse(is_prime(-1))
        self.assertFalse(is_prime(-2))
        self.assertFalse(is_prime(-3))
        self.assertFalse(is_prime(-4))
        self.assertFalse(is_prime(-5))
        self.assertFalse(is_prime(-6))
        self.assertFalse(is_prime(-7))
        self.assertFalse(is_prime(-8))
        self.assertFalse(is_prime(-9))

这两个是完全等价的。除了当我们运行循环版本时，我们得到了一个我们不太想要的信息：

    
    
    python test_primes.py
    ...F
    ======================================================================
    FAIL: test_negative_number (__main__.PrimesTestCase)
    Is a negative number correctly determined not to be prime?
    ----------------------------------------------------------------------
    Traceback (most recent call last):
    File "test_primes.py", line 22, in test_negative_number
        self.assertFalse(is_prime(index))
    AssertionError: True is not false
    ----------------------------------------------------------------------
    Ran 4 tests in 0.000s
    FAILED (failures=1)

嗯···我们知道测试失败了，但是是在哪个负数上失败的？非常没用的是，Python的单元测试框架并没有打印出预期值和实际值。我们可以移步到两种方式上，并用其中
之一来解决问题：通过msg参数，或通过使用一个第三方的单元测试框架。

使用msg参数来assertFalse仅仅能够使我们认识到我们可以用字符串的格式设置来解决问题。

    
    
    def test_negative_number(self):
        """Is a negative number correctly determined not to be prime?"""
        for index in range(-1, -10, -1):
            self.assertFalse(is_prime(index), msg='{} should not be determined to be prime'.format(index))

从而给出了如下输出信息：

    
    
    python test_primes
    ...F
    ======================================================================
    FAIL: test_negative_number (test_primes.PrimesTestCase)
    Is a negative number correctly determined not to be prime?
    ----------------------------------------------------------------------
    Traceback (most recent call last):
    File "./test_primes.py", line 22, in test_negative_number
        self.assertFalse(is_prime(index), msg='{} should not be determined to be prime'.format(index))
    AssertionError: True is not false : -1 should not be determined to be prime
    ----------------------------------------------------------------------
    Ran 4 tests in 0.000s
    FAILED (failures=1)

### 妥善地修复代码

我们看到，失败的负数是第一个数字：-1。为了解决这个问题，我们可以为负数增再增加一个特殊检查，但是编写单元测试的目的不是盲目的添加代码来检测边界情况。当一个
测试失败时，我们应该退后一步并且确定解决问题的最佳方式。在这种情况下，我们就不该增加一个额外的if：

    
    
    def is_prime(number):
        """Return True if *number* is prime."""
        if number < 0:
            return False
        if number in (0, 1):
            return False
        for element in range(2, number):
            if number % element == 0:
                return False
        return True

应当首先使用如下代码：

    
    
    def is_prime(number):
        """Return True if *number* is prime."""
        if number <= 1:
            return False
        for element in range(2, number):
            if number % element == 0:
                return False
        return True

在后一个代码中，我们发现如果参数小于等于1时，两个if语句可以合并到一个返回值为false的语句中。这样做不仅更加简洁，并且很好的贴合了素数的定义（一个比1
大并且只能被1和它本身整除的数）。

### 第三方测试框架

我们本来也可以通过使用第三方测试框架解决这个由于信息太少导致测试失败的问题。最常用的两个是py.test和nose。通过运行语句py.test
-l（-l为显示局部变量的值）可以得到如下结果。

    
    
    #! bash
    py.test -l test_primes.py
    ============================= test session starts ==============================
    platform linux2 -- Python 2.7.6 -- pytest-2.4.2
    collected 4 items
    test_primes.py ...F
    =================================== FAILURES ===================================
    _____________________ PrimesTestCase.test_negative_number ______________________
    self =     def test_negative_number(self):
            """Is a negative number correctly determined not to be prime?"""
            for index in range(-1, -10, -1):
    >           self.assertFalse(is_prime(index))
    E           AssertionError: True is not false
    index      = -1
    self       = test_primes.py:22: AssertionError

正如你所看到的，一些更有用的信息。这些框架提供了比单纯的更详细的输出更多的功能，但问题是仅仅知道它们能存在和扩展内置unittest测试包的功能。

结束语

在这篇文章中，你学到了什么是单元测试，为什么它们如此重要，还有怎样编写测试。这就是说，要注意我们只是剖开了测试方法学中的表层，更多高级的话题，比如测试案例的
组织、持续整合以及测试案例的管理等都是可供那些想要进一步学习Python中的测试的读者研究的很好的话题。

在不改变其功能的前提下重组/清理代码

编代码时不暴露其内部数据或函数并且不使用其他代码的内部数据或函数

  

文章转自：<http://blog.jobbole.com/55180/> 作者：卷卷怪

英文出处：http://jeffknupp.com/blog/2013/12/09/improve-your-python-understanding-
unit-testing/

