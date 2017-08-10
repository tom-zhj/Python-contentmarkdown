# python代码制作configure文件

在lua中，一直用lua作为config文件，或承载数据的文件 -
好处是lua本身就很好阅读，然后无需额外写解析的代码，还支持在configure文件中读环境变量，条件判断等。

在lua中通过loadfile, setfenv实现）

python：

    
    
    cat config.py
    bar = 10
    foo=100
    cat python_as_config.py:
    ns = {}
    execfile('config.py', ns)
    print "\n".join(sorted(dir(ns)))
    print "*"*80
    print ns['foo']
    print ns['bar']

缺点是不像lua那么可以以成员的方式访问table中的变量，如ns.foo, ns.bar...

  

