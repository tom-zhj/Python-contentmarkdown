# Python连接Redis连接配置

系统环境：

OS：Oracle Linux Enterprise 5.6

redis：redis-2.6.8

python：Python-2.7.3

redis的python包版本：redis-2.7.2.tar

前提条件：

1.确保Redis已成功安装并且正确配置，参考文档

2.确保Python环境已成功配置，参考文档

配置python连接redis:

1.安装Redis的Python包：

使用easy-install安装，关于easy-install的配置，参考以上Python环境的搭建。

[root@njdyw bin]# easy_install2.7.3 redis

Searching for redis

Reading http://pypi.python.org/simple/redis/

Reading http://github.com/andymccurdy/redis-py

Best match: redis 2.7.2

Downloading http://pypi.python.org/packages/source/r/redis/redis-2.7.2.tar.gz#
md5=17ac60dcf13eb33f82cc25974ab17157

Processing redis-2.7.2.tar.gz

Running redis-2.7.2/setup.py -q bdist_egg --dist-dir /tmp/easy_install-
8FAlft/redis-2.7.2/egg-dist-tmp-JzQViJ

zip_safe flag not set; analyzing archive contents...

Adding redis 2.7.2 to easy-install.pth file

Installed /usr/local/python2.7.3/lib/python2.7/site-
packages/redis-2.7.2-py2.7.egg

Processing dependencies for redis

Finished processing dependencies for redis

\--安装Parser包（可选）

说明：Parser可以控制如何解析redis响应的内容。redis-
py包含两个Parser类，PythonParser和HiredisParser。默认，如果已经安装了hiredis模块，redis-
py会使用HiredisParser，否则会使用PythonParser。

HiredisParser是C编写的，由redis核心团队维护，性能要比PythonParser提高10倍以上，所以推荐使用。安装方法，使用easy_ins
tall：

[root@njdyw ~]# easy_install2.7.3 hiredis

Searching for hiredis

Reading http://pypi.python.org/simple/hiredis/

Reading https://github.com/pietern/hiredis-py

Best match: hiredis 0.1.1

Downloading http://pypi.python.org/packages/source/h/hiredis/hiredis-0.1.1.tar
.gz#md5=92128474f6fb027cfb8587fce724ea8e

Processing hiredis-0.1.1.tar.gz

Running hiredis-0.1.1/setup.py -q bdist_egg --dist-dir /tmp/easy_install-
ZanSCB/hiredis-0.1.1/egg-dist-tmp-XCZBQ0

zip_safe flag not set; analyzing archive contents...

Adding hiredis 0.1.1 to easy-install.pth file

Installed /usr/local/python2.7.3/lib/python2.7/site-
packages/hiredis-0.1.1-py2.7-linux-x86_64.egg

Processing dependencies for hiredis

Finished processing dependencies for hiredis

2.检查安装是否成功

\--easy-install安装的扩展包默认在python的site-packages目录下

[root@njdyw ~]#whereis python2.7.3

python2.7: /bin/python2.7.3 /usr/local/python2.7.3

[root@njdyw ~]#cd /usr/local/python2.7.3/lib/python2.7/site-packages/

[root@njdyw site-packages]# ll

总计 408

-rw-r--r-- 1 root root    239 03-21 10:45 easy-install.pth

-rw-r--r-- 1 root root    119 03-21 10:07 README

-rw-r--r-- 1 root root  60401 03-21 10:45redis-2.7.2-py2.7.egg

-rw-r--r-- 1 root root 332125 03-21 10:12 setuptools-0.6c11-py2.7.egg

-rw-r--r-- 1 root root     30 03-21 10:12 setuptools.pth

可以看到redis-2.7.2-py2.7.egg包已经成功安装

3.测试连接

[root@njdyw site-packages]#python2.7.3

Python 2.7.3 (default, Mar 21 2013, 10:06:48)

[GCC 4.1.2 20080704 (Red Hat 4.1.2-50)] on linux2

Type "help", "copyright", "credits" or "license" for more information.

>>>import redis

>>>redisClient=redis.StrictRedis(host='127.0.0.1',port=6379,db=0)

>>> redisClient.set('test_redis','Hello Python')

True

>>> value=redisClient.get('test_redis')

>>> print value

Hello Python

>>> redisClient.delete('test_redis')

True

>>> value=redisClient.get('test_redis')

>>> print value

None

>>> dir(redis)

['AuthenticationError', 'Connection', 'ConnectionError', 'ConnectionPool',
'DataError', 'InvalidResponse', 'PubSubError', 'Redis', 'RedisError',
'ResponseError', 'StrictRedis', 'UnixDomainSocketConnection', 'VERSION',
'WatchError', '__all__', '__builtins__', '__doc__', '__file__', '__loader__',
'__name__', '__package__', '__path__', '__version__', '_compat', 'client',
'connection', 'exceptions', 'from_url', 'utils']

>>> redisClient=redis.StrictRedis(host='127.0.0.1',port=6379,db=0)

>>> dir(redisClient)

['RESPONSE_CALLBACKS', '__class__', '__contains__', '__delattr__',
'__delitem__', '__dict__', '__doc__', '__format__', '__getattribute__',
'__getitem__', '__hash__', '__init__', '__module__', '__new__', '__reduce__',
'__reduce_ex__', '__repr__', '__setattr__', '__setitem__', '__sizeof__',
'__str__', '__subclasshook__', '__weakref__', '_zaggregate', 'append',
'bgrewriteaof', 'bgsave', 'bitcount', 'bitop', 'blpop', 'brpop', 'brpoplpush',
'client_kill', 'client_list', 'config_get', 'config_set', 'connection_pool',
'dbsize', 'debug_object', 'decr', 'delete', 'echo', 'eval', 'evalsha',
'execute_command', 'exists', 'expire', 'expireat', 'flushall', 'flushdb',
'from_url', 'get', 'getbit', 'getrange', 'getset', 'hdel', 'hexists', 'hget',
'hgetall', 'hincrby', 'hincrbyfloat', 'hkeys', 'hlen', 'hmget', 'hmset',
'hset', 'hsetnx', 'hvals', 'incr', 'incrbyfloat', 'info', 'keys', 'lastsave',
'lindex', 'linsert', 'llen', 'lock', 'lpop', 'lpush', 'lpushx', 'lrange',
'lrem', 'lset', 'ltrim', 'mget', 'move', 'mset', 'msetnx', 'object',
'parse_response', 'persist', 'pexpire', 'pexpireat', 'ping', 'pipeline',
'pttl', 'publish', 'pubsub', 'randomkey', 'register_script', 'rename',
'renamenx', 'response_callbacks', 'rpop', 'rpoplpush', 'rpush', 'rpushx',
'sadd', 'save', 'scard', 'script_exists', 'script_flush', 'script_kill',
'script_load', 'sdiff', 'sdiffstore', 'set', 'set_response_callback',
'setbit', 'setex', 'setnx', 'setrange', 'shutdown', 'sinter', 'sinterstore',
'sismember', 'slaveof', 'smembers', 'smove', 'sort', 'spop', 'srandmember',
'srem', 'strlen', 'substr', 'sunion', 'sunionstore', 'time', 'transaction',
'ttl', 'type', 'unwatch', 'watch', 'zadd', 'zcard', 'zcount', 'zincrby',
'zinterstore', 'zrange', 'zrangebyscore', 'zrank', 'zrem', 'zremrangebyrank',
'zremrangebyscore', 'zrevrange', 'zrevrangebyscore', 'zrevrank', 'zscore',
'zunionstore']

>>>

4.测试实例：

(1).把文本数据导入到redis

\--导入的数据格式

[root@njdyw ~]#more data.txt

wolys # wolysopen111 # wolys@21cn.com

coralshanshan # 601601601 # zss1984@126.com

pengfeihuchao # woaidami # 294522652@qq.com

simulategirl # @#$9608125 # simulateboy@163.com

daisypp # 12345678 # zhoushigang_123@163.com

sirenxing424 # tfiloveyou # sirenxing424@126.com

raininglxy # 1901061139 # lixinyu23@qq.com

leochenlei # leichenlei # chenlei1201@gmail.com

z370433835 # lkp145566 # 370433835@qq.com

\--创建命令脚本

[root@njdyw ~]#cat imp_red.py

import redis

import re

pool = redis.ConnectionPool(host='127.0.0.1', port=6379)

r = redis.Redis(connection_pool=pool)

pipe = r.pipeline()

p=re.compile(r'(.*)\s#\s(.*)\s#\s(.*)');

pipe = r.pipeline()

f = open("data.txt")

matchs=p.findall(f.read())

for user in matchs:

  key='users_%s' %user[0].strip()

  pipe.hset(key,'pwd',user[1].strip()).hset(key,'email',user[2].strip())

pipe.execute()

f.close()

注意：要严格控制python脚本中的空格

\--执行脚本

[root@njdyw ~]# python2.7.3 imp_red.py

\--查看导入数据

[root@njdyw ~]#redis-cli

redis 127.0.0.1:6379> keys *

1) "users_xiaochuan2018"

2) "users_coralshanshan"

3) "users_xiazai200901"

4) "users_daisypp"

5) "users_boiny"

6) "users_raininglxy"

7) "users_fennal"

8) "users_abc654468252"

9) "users_babylovebooks"

10) "users_xl200811"

11) "users_baby19881018"

12) "users_darksoul0929"

13) "users_pengcfwxh"

14) "users_alex126126"

15) "users_jiongjiongmao"

16) "users_sirenxing424"

17) "users_mengjie007"

18) "users_cxx0409"

19) "users_candly8509"

20) "users_licaijun007"

21) "users_ai3Min2"

22) "users_bokil"

23) "users_z370433835"

24) "users_yiling1007"

25) "users_simulategirl"

26) "users_fxh852"

27) "users_baoautumn"

28) "users_huangdaqiao"

29) "users_q1718334567"

30) "users_xldq_l"

31) "users_beibeilong012"

32) "users_hudaoyin"

33) "users_yoyomika"

34) "users_jacksbalu"

35) "users_wolys"

36) "users_kangte1"

37) "users_demonhaodh"

38) "users_ysdz8"

39) "users_leochenlei"

40) "users_llx6888"

41) "users_pengfeihuchao"

redis 127.0.0.1:6379>

redis 127.0.0.1:6379>hget users_pengfeihuchao email

"294522652@qq.com"

redis 127.0.0.1:6379> hget users_llx6888 email

"linlixian200606@126.com"

  

