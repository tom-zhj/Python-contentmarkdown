# Python代码规范利器Flake8

写代码其实是需要规范的，团队中更是如此；不然 Google 也不会发布各种编码规范，耳熟能详的有Google C++ 风格指南，Google Python
风格指南，等等。

这些规范有用吗？有用也没用，除非你脑子好使，一边 coding，一边将规范运用的发紫；否则我们终须还是需要一种工具来做这件事情。好在python
不止一种工具帮我们做这件事。话休絮烦，切正题。

Pylint

使用过，变态到发紫；不知道谁那么无聊，将规则定的那么死，我们 pythoner 能快乐吗？乃们不见 rubyer，Matz 倡导的是什么？ Happy
Coding 有木有？ 所以用过就仍了，因为我不需要这么变态的搞，无爱～ 如果你要安装，也很简单：

<php>easy_install pylint // maybe nedd root</php>

Pep8

顾名思义，来自于 Python 社区著名的 PEP 8。基本上写代码按这个就对了，但是这还不够完美；安装如下：

easy_install pep8 // maybe nedd root

  

Pyflakes

  

Python 程序被动检测工具，还真够被动的，据作者说比较快，不够强大，但是还可以～

easy_install pyflakes // maybe nedd root

Flake8

  

主角登场了，这是我推荐的，但是并不影响其他人喜欢 pylint。其实这哥们是集大成者，是以下三个工具的包装：

PyFlakes

Pep8

Ned Batchelder’s McCabe script

好处不说了，关键是可扩展的，这儿说的很清楚了：https://pypi.python.org/pypi/flake8/2.0。安装如下：

easy_install flake8 // maybe nedd root

如果你跟我一样喜欢 Git 这丫，那么你也是 pyhoner，那么还有福利，将如下的代码写入 .git/hooks/pre-commit：

#!/usr/bin/env python

import sys

from flake8.hooks import git_hook

  

COMPLEXITY = 12

STRICT = True

  

代码就不解释了，官方文档写的很清楚：http://flake8.readthedocs.org/en/latest/vcs.html#git-hook。
如果你的 pre-commit 脚本已经有了规则，也没事，在 shell 中调用 python 吧。

如果你也喜欢 vim

  

作为两大神器之一的 vim，自然要有插件来享受以上工具的：

nvie/vim-flake8

vim-scripts/pylint.vim

看到上面的列举，你应该会知道我在说什么了，没错，用 vundle 安装：

" Flake8 plugin for Vim.

Bundle 'nvie/vim-flake8'

" compiler plugin for python style checking tool.

Bundle 'vim-scripts/pylint.vim'

autocmd FileType python compiler pylint

  

如果你真的不知道 vundle，真的是时候使用她了：https://github.com/gmarik/vundle。如果你嫌这一切都麻烦，直接用我的
vimrc 吧，在这里：

  

git clone https://github.com/icocoa/icocoa-vimrc.git --recursive vimrc //
icocoa is my another account in GitHub

  

