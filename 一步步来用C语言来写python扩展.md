# 一步步来用C语言来写python扩展

本文介绍如何用 C 语言来扩展 python。所举的例子是，为 python 添加一个设置字符串到 windows
的剪切板(Clipboard)的功能。我在写以下代码的时候用到的环境是：windows xp, gcc.exe 4.7.2, Python 3.2.3。

第一步 撰写C语言的DLL

创建一个 clip.c 文件，内容如下：

    
    
    // 设置 UNICODE 库，这样的话才可以正确复制宽字符集
    #define UNICODE
     
    #include <windows.h>
    #include <python.h>
     
    // 设置文本到剪切板(Clipboard)
    static PyObject *setclip(PyObject *self, PyObject *args)
    {
      LPTSTR  lptstrCopy;
      HGLOBAL hglbCopy;
      Py_UNICODE *content;
      int len = 0;
     
      // 将 python 的 UNICODE 字符串及长度传入
      if (!PyArg_ParseTuple(args, "u#", &content, &len))
        return NULL;
     
      Py_INCREF(Py_None);
     
      if (!OpenClipboard(NULL))
        return Py_None;
     
      EmptyClipboard();
     
      hglbCopy = GlobalAlloc(GMEM_MOVEABLE, (len+1) * sizeof(Py_UNICODE));
      if (hglbCopy == NULL) {
        CloseClipboard();
        return Py_None;
      }
     
      lptstrCopy = GlobalLock(hglbCopy);
      memcpy(lptstrCopy, content, len * sizeof(Py_UNICODE));
      lptstrCopy[len] = (Py_UNICODE) 0;
     
      GlobalUnlock(hglbCopy);
     
      SetClipboardData(CF_UNICODETEXT, hglbCopy);
     
      CloseClipboard();
     
      return Py_None;
    }
     
    // 定义导出给 python 的方法
    static PyMethodDef ClipMethods[] = {
      {"setclip", setclip, METH_VARARGS,
       "Set string to clip."},
      {NULL, NULL, 0, NULL}
    };
     
    // 定义 python 的 model
    static struct PyModuleDef clipmodule = {
      PyModuleDef_HEAD_INIT,
      "clip",
      NULL,
      -1,
      ClipMethods
    };
     
    // 初始化 python model
    PyMODINIT_FUNC PyInit_clip(void)
    {
      return PyModule_Create(&clipmodule);
    }

第二步 写 python 的 setup.py

创建一个 setup.py 文件，内容如下：

    
    
    from distutils.core import setup, Extension
     
    module1 = Extension('clip',
                        sources = ['clip.c'])
     
    setup (name = 'clip',
           version = '1.0',
           description = 'This is a clip package',
           ext_modules = [module1])

第三步 用 python 编译

运行以下命令：

python setup.py build --compiler=mingw32 install

在我的环境中会提示以下错误：

gcc: error: unrecognized command line option '-mno-cygwin'

error: command 'gcc' failed with exit status 1

打开 %PYTHON安装目录%/Lib/distutils/cygwinccompiler.py 文件，将里面的 -mno-cygwin
删除掉，然后再运行即可。

  

正常运行后，会生成一个 clip.pyd 文件，并将该文件复制到 %PYTHON安装目录%/Lib/site-packages 目录中

  

第四步 测试该扩展

写一个 test.py, 内容如下：

    
    
    # -*- encoding: gbk -*-
    import clip
    clip.setclip("Hello python")

运行

python test.py

再到任何一个地方粘贴，即可验证是否正确。

  

