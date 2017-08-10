# 解决ImportError: libmysqlclient_r.so.16: cannot open shared object file

在开发一个python项目是，需要用到mysql，但是，

安装完mysql-python后import加载模块提示以下错误：

ImportError: libmysqlclient_r.so.16: cannot open shared object file: No such
file or directory

可以尝试一下两种方法：

方法一：

在mysql-ython的安装目录下找到site.cfg，将

#mysql_config = XXXXXXXXXXXXXXXX

注释符号去掉，并填上mysql_config的地址

  

方法二：

将mysql/lib下所有关于libmysqlclient的so文件软链接到/usr/lib下。

>>> ln -s /usr/local/mysql/lib/mysql/libmysqlclient* /usr/lib

重新加载配置

>>> ldconfig

这时候就不会出错了

  

