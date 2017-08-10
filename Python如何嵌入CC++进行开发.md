# Python如何嵌入C/C++进行开发

如果你想把Python嵌入C/C++中是比较简单的事情，你需要的是在VC中添加Python的include文件目录和lib文件目录。下面我们来看下如何把Py
thon嵌入C/C++中。

VC6.0下，打开 tools->options->directories->show directories
for，将Python安装目录下的inlude目录添加到inlude files项中，将libs目录添加到library files项中。

VC2005下，打开tools->options->项目和解决方案->VC++目录，然后做相同工作。

代码如下：

在debug下执行出错，“无法找到python31_d.lib文件”，后查到原因是：在debug下生成必须要有python31_d.lib文件，否则只能在r
elease下生成

    
    
    #include <python.h> 
    int main()  
    {  
    Py_Initialize();  
    PyRun_SimpleString("Print 'hi, python!'");  
    Py_Finalize();  
    return 0;  
    }

Py_Initialize函数原型是：void Py_Initialize()

把Python嵌入C/C++中时必须使用该函数，它初始化Python解释器，在使用其他的Python/C
API之前必须先调用该函数。可以使用Py_IsInitialized函数判断是否初始化成功，成功返回True。

PyRun_SimpleString函数原型是int PyRun_SimpleString(const char
*command)，用来执行一段Python代码。

注意：是否需要维持语句间的缩进呢？

Py_Finalize函数原型是void Py_Finalize()，用于关闭Python解释器，释放解释器所占用的资源。

PyRun_SimpleFile函数可以用来运行".py"脚本文件，函数原型如下：

int PyRun_SimpleFile(FILE *fp, const char *filename);

其 中fp是打开的文件指针，filename是要运行的python脚本文件名。但是由于该函数官方发布的是由visual studio
2003.NET编译的，如果使用其他版本的编译器，FILE定义可能由于版本原因导致崩溃。同时，为简便起见可以使用如下方式来代替该函数：

PyRun_SimpleString("execfile(‘file.py’)"); //使用execfile来运行python文件

Py_BuildValue()用于对数字和字符串进行转换处理，变成Python中相应的数据类型（在C语言中，所有Python类型都被声明为PyObject类
型），函数原型如下：

PyObject *Py_BuildValue(const char *format, …..);

PyString_String()用于将PyObject*类型的变量转换成C语言可以处理的char*型，具体原型如下：

char* PyString_String(PyObject *p);

以上就是对如何把Python嵌入C/C++中相关的内容的介绍，如果有问题欢迎在下方留言。

  

