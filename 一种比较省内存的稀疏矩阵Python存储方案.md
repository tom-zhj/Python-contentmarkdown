# 一种比较省内存的稀疏矩阵Python存储方案

推荐系统中经常需要处理类似user_id, item_id, rating这样的数据，其实就是数学里面的稀疏矩阵，scipy中提供了sparse模块来解决这
个问题，但scipy.sparse有很多问题不太合用：1、不能很好的同时支持data[i, ...]、data[..., j]、data[i,
j]快速切片；2、由于数据保存在内存中，不能很好的支持海量数据处理。

要支持data[i, ...]、data[..., j]的快速切片，需要i或者j的数据集中存储；同时，为了保存海量的数据，也需要把数据的一部分放在硬盘上，用
内存做buffer。这里的解决方案比较简单，用一个类Dict的东西来存储数据，对于某个i（比如9527），它的数据保存在dict['i9527']里面，同样
的，对于某个j（比如3306），它的全部数据保存在dict['j3306']里面，需要取出data[9527, ...]的时候，只要取出dict['i952
7']即可，dict['i9527']原本是一个dict对象，储存某个j对应的值，为了节省内存空间，我们把这个dict以二进制字符串形式存储，直接上代码：

    
    
    '''
    Sparse Matrix
    '''
    import struct
    import numpy as np
    import bsddb
    from cStringIO import StringIO
     
    class DictMatrix():
        def __init__(self, container = {}, dft = 0.0):
            self._data  = container
            self._dft   = dft
            self._nums  = 0
     
        def __setitem__(self, index, value):
            try:
                i, j = index
            except:
                raise IndexError('invalid index')
     
            ik = ('i%d' % i)
            # 为了节省内存，我们把j, value打包成字二进制字符串
            ib = struct.pack('if', j, value)
            jk = ('j%d' % j)
            jb = struct.pack('if', i, value)
     
            try:
                self._data[ik] += ib
            except:
                self._data[ik] = ib
            try:
                self._data[jk] += jb
            except:
                self._data[jk] = jb
            self._nums += 1
     
        def __getitem__(self, index):
            try:
                i, j = index
            except:
                raise IndexError('invalid index')
     
            if (isinstance(i, int)):
                ik = ('i%d' % i)
                if not self._data.has_key(ik): return self._dft
                ret = dict(np.fromstring(self._data[ik], dtype = 'i4,f4'))
                if (isinstance(j, int)): return ret.get(j, self._dft)
     
            if (isinstance(j, int)):
                jk = ('j%d' % j)
                if not self._data.has_key(jk): return self._dft
                ret = dict(np.fromstring(self._data[jk], dtype = 'i4,f4'))
     
            return ret
     
        def __len__(self):
            return self._nums
     
        def __iter__(self):
            pass
     
        '''
        从文件中生成matrix
        考虑到dbm读写的性能不如内存，我们做了一些缓存，每1000W次批量写入一次
        考虑到字符串拼接性能不太好，我们直接用StringIO来做拼接
        '''
        def from_file(self, fp, sep = 't'):
            cnt = 0
            cache = {}
            for l in fp:
                if 10000000 == cnt:
                    self._flush(cache)
                    cnt = 0
                    cache = {}
                i, j, v = [float(i) for i in l.split(sep)]
     
                ik = ('i%d' % i)
                ib = struct.pack('if', j, v)
                jk = ('j%d' % j)
                jb = struct.pack('if', i, v)
     
                try:
                    cache[ik].write(ib)
                except:
                    cache[ik] = StringIO()
                    cache[ik].write(ib)
     
                try:
                    cache[jk].write(jb)
                except:
                    cache[jk] = StringIO()
                    cache[jk].write(jb)
     
                cnt += 1
                self._nums += 1
     
            self._flush(cache)
            return self._nums
     
        def _flush(self, cache):
            for k,v in cache.items():
                v.seek(0)
                s = v.read()
                try:
                    self._data[k] += s
                except:
                    self._data[k] = s
     
    if __name__ == '__main__':
        db = bsddb.btopen(None, cachesize = 268435456)
        data = DictMatrix(db)
        data.from_file(open('/path/to/log.txt', 'r'), ',')

测试4500W条rating数据（整形,整型,浮点格式），922MB文本文件导入，采用内存dict储存的话，12分钟构建完毕，消耗内存1.2G，采用示例代码
中的bdb存储，20分钟构建完毕，占用内存300～400MB左右，比cachesize大不了多少，数据读取测试：

    
    
    import timeit
    timeit.Timer('foo = __main__.data[9527, ...]', 'import __main__').timeit(number = 1000)

消耗1.4788秒，大概读取一条数据1.5ms。

采用类Dict来存储数据的另一个好处是你可以随便用内存Dict或者其他任何形式的DBM，甚至传说中的Tokyo Cabinet….

好的，码完收工。

  

