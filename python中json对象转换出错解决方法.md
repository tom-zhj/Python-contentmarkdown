# python中json对象转换出错解决方法

今天在使用**python中的json转换**碰到一个问题:

接收一个post的json字符串:

s={"username":"admin","password":"password","tenantid":""}

使用python自带的json库

    
    
    import json
    >>> a=json.loads(s)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "/usr/lib/python2.7/json/__init__.py", line 326, in loads
        return _default_decoder.decode(s)
      File "/usr/lib/python2.7/json/decoder.py", line 366, in decode
        obj, end = self.raw_decode(s, idx=_w(s, 0).end())
    TypeError: expected string or buffer
    >>>

  

出错！

百思不得其解。经过调试，最终发现，python中默认使用单引号表示字符串"'"

所以当，用字符串符值以后，python会把双引号转换为单引号

>>> s={"username":"admin","password":"password","tenantid":""}

>>> print s

{'username': 'admin', 'password': 'password', 'tenantid': ''}

而json是不支持单引号的。

可以用下面的方法转换

json_string=json.dumps(s)

python_obj=json.loads(json_string)

  

ok，问题解决

