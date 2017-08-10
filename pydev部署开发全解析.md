# pydev部署开发全解析

把pydev开发的一个上传项目部署到测试环境时

1、提示找不到我写的模块

解决方法：在项目入口增加代码

#在项目的PYTHONPATH 添加源目录

PROJECT_DIR = os.path.dirname(__file__)

PROJECT_ROOT_DIR = os.path.dirname(PROJECT_DIR)

if not PROJECT_ROOT_DIR in sys.path:

sys.path.append(PROJECT_ROOT_DIR)

if not PROJECT_DIR in sys.path:

sys.path.append(PROJECT_DIR)

2、运行时提示默认编码为ascii，不是utf-8错误

解决方法：在python的site-packages目录下增加sitecustomize.py

import sys

sys.setdefaultencoding("utf-8")

3、easy_install安装PIL时提示不支持jpeg（png类似）

解决方法：由于centos是64位的，在yum install libjpeg* 时，安装到了/usr/lib64目录下，

而PIL默认搜索的路径是/usr/lib，所以未找到而报错，直接做ln即可

  

