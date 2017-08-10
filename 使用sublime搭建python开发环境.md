# 使用sublime搭建python开发环境

sublime Text具有漂亮的用户界面和强大的功能，例如代码缩略图，Python的插件，代码段等。还可自定义键绑定，菜单和工具栏。Sublime
Text的主要功能包括：拼写检查，书签，完整的 Python API，Goto功能，即时项目切换，多选择，多窗口等等。

Step1：安装python和sublime

Step2：给sublime安装package control，安装参见： 官网

Step3：配置安装路径

方式一：配置windows的Path

好处就是cmd的时候也可以运行，视为系统，用户级别的配置；

方式二：配置sublime的python的sublime_build

点击：Preference -> Browse Packages -> 在python目录下,编辑Python.sublime-
build文件，添加python应用程序的路径：

    
    
    {
     "cmd":["python.exe", "-u", "$file"],
     "path":"C:/Python27",
     "file_regex": "^[ ]*File \"(...*?)\", line ([0-9]*)",
     "selector": "source.python"
    }

  

至此，就可以使用ctrl ＋ b的方式直接运行当前的python代码了。

step4：安装某些插件

安装方式：

Ctrl+Shift+P打开控制面板，找到Install Package，回车；在弹出框中输入插件名称,回车安装即可。或者按照插件的说明文档进行安装。

网上推荐插件：

SublimeREPL 可以用于运行和调试一些需要交互的程序。

SublimeCodeIntel 可以支持代码的自动补全以及成员/方法提示等功能。

SublimeLinter 是用来在写代码时做代码检查的，可以检查Python代码是否符合PEP8的要求。

  

