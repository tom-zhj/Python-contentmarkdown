# 注意for循环中变量的作用域


    for e in collections:
        pass

在for 循环里， 最后一个对象e一直存在在上下文中。就是在循环外面，接下来对e的引用仍然有效。

这里有个问题容易被忽略，如果在循环之前已经有一个同名对象存在，这个对象是被覆盖的。

如果在有代码感知的IDE中， IDE会提示变量是“被重新声明的”， 但运行时却不会出错。

for循环不是闭包，可以使用dis模块分解以下代码可以看到：

    
    
    x = 5
    for x in range(10):
        pass
    print x

将代码保存到test.py文件，运行python -m dis test.py

  

C:\Users\Patrick\Desktop>python -m dis test.py

  1           0 LOAD_CONST               0 (5)

              3 STORE_NAME               0 (x)

  

  3           6 SETUP_LOOP              20 (to 29)

              9 LOAD_NAME                1 (range)

             12 LOAD_CONST               1 (10)

             15 CALL_FUNCTION            1

             18 GET_ITER

        >>   19 FOR_ITER                 6 (to 28)

             22 STORE_NAME               0 (x)

  

  4          25 JUMP_ABSOLUTE           19

        >>   28 POP_BLOCK

  

  6     >>   29 LOAD_NAME                0 (x)

             32 PRINT_ITEM

             33 PRINT_NEWLINE

             34 LOAD_CONST               2 (None)

             37 RETURN_VALUE

在其他语言里，for循环的初始化变量对于上下文同样是可见的，比如java， 因为java是强类型的语言， 如果重新声明已存在的变量IDE会提示错误，
当然不同通过编译。

通常在python编程中(可能是大多数的动态语言)，有时即使声明了同名的变量，程序没有出现明显的错误，但是一旦出错，错误很难被发现。所以要避免与for循环中
的变量重名。

在使用python模板语言编码时尤其如此。代码编辑器没有提示，不会发现错误在哪里。这个是我碰到的极其怪异的一个例子。为什么说怪异，因为逻辑上没有任何问题。

在一个页面模板里面，当handler调用这个模板时，同时传递了两个对象（从handler中，我使用tornado），一个page对象和一个pages列表。我
的顺序是这样的：

<!-- 用page对象 -->

<label>{{ page.name if page else ''}}</label>

<!-- 用pages对象 -->

<label>Parent Page

    <select name="parent_id">

        {% if pages %}

            {% for page in pages%}

            <option value="{{ page.id}}">{{page.name}}</option>

            {% end %}

        {% end %}

        <option value="">None</option>

    </select>

</label>

  

<!-- 然后又page -->

<div>{{ page.markdown if page else ''}}</div>

问题来了，在运行的时候出错了，提示在 <label>{{ page.name if page else ''}}</label> 中错误page
referenced before assignment.

晕死了， 找了一晚上的错，最后在把for循环中page的名字改为_page才运行了。

在模板调用过程里，模板语言也是被翻译到python字节码，并按行解析和出，所以根本没有逻辑，不知道是tornado模板语言的bug。

所以注意变量名。

总之我认为tornado的exception trace非常不友好。

Python中变量的作用域搜索顺序：本地作用域（Local）→当前作用域被嵌入的本地作用域（Enclosing
locals）→全局/模块作用域（Global）→内置作用域（Built-in）

