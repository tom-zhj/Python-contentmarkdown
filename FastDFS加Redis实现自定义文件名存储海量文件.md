# FastDFS加Redis实现自定义文件名存储海量文件

FastDFS非常适合存储大量的小文件，遗憾的是本身不支持自定义文件名，文件名是存储成功以后根据存储位置生成的一个file_id。很多应用场景不得不使用自定
义文件名，在不修改其源码的情况下，可以在存储客户端fdfs_client增加一个用来存储自定义文件名和fastdfs的file_id之间的映射关系的数据库间
接实现自定义文件名的存取和访问，在这里我们选用了reids。顺便说一下，淘宝也有一个类似于FastDFS的文件存储系统TFS，对于自定义文件名，它是用mys
ql来存储映射关系的，我认为在高并发访问下mysql本身就是瓶颈，因此在这个方案中采用了redis。

准备工作：

fastdfs环境安装...略...（官方：https://code.google.com/p/fastdfs/）

redis环境安装...略...(官方：http://redis.io/)

用python实现，因此需要安装fastdfs的python客户端(下载：https://fastdfs.googlecode.com/files
/fdfs_client-py-1.2.6.tar.gz)

python的redis客户端，到https://pypi.python.org/pypi/redis下载

    
    
    # -*- coding: utf-8 -*-
    import setting
    from fdfs_client.client import *
    from fdfs_client.exceptions import *
     
    from fdfs_client.connection import *
     
    import redis
    import time
    import logging
    import random
     
    logging.basicConfig(format='[%(levelname)s]: %(message)s', level=logging.DEBUG)
    logger = logging.getLogger(__name__)
    logger.setLevel(logging.DEBUG)
     
     
    class RedisError(Exception):
         def __init__(self, value):
             self.value = value
         def __str__(self):
             return repr(self.value)
     
    class fastdfsClient(Fdfs_client):
        def __init__(self):
            self.tracker_pool = ConnectionPool(**setting.fdfs_tracker)
            self.timeout  = setting.fdfs_tracker['timeout']
            return None
     
        def __del__(self):
            try:
                self.pool.destroy()
                self.pool = None
            except:
                pass
     
    class fastdfs(object):
        def __init__(self):
            '''
            conf_file:配置文件
            '''
            self.fdfs_client = fastdfsClient()
            self.fdfs_redis = []
            for i in setting.fdfs_redis_dbs:
                self.fdfs_redis.append(redis.Redis(host=i[0], port=i[1], db=i[2]))
     
        def store_by_buffer(self,buf,filename=None,file_ext_name = None):
            '''
            buffer存储文件
            参数：
            filename:自定义文件名，如果不指定，将远程file_id作为文件名
            file_ext_name:文件扩展名（可选），如果不指定，将根据自定义文件名智能判断
            返回值：
            {
            'group':组名,
            'file_id':不含组名的文件ID,
            'size':文件尺寸,
            'upload_time':上传时间
            }
            '''
            if filename and  random.choice(self.fdfs_redis).exists(filename):
                logger.info('File(%s) exists.'%filename)
                return   random.choice(self.fdfs_redis).hgetall(filename)
            t1 = time.time()
    #        try:
            ret_dict = self.fdfs_client.upload_by_buffer(buf,file_ext_name)
    #        except Exception,e:
    #            logger.error('Error occurred while uploading: %s'%e.message)
    #            return None
            t2 = time.time()
            logger.info('Upload file(%s) by buffer, time consume: %fs' % (filename,(t2 - t1)))
            for key in ret_dict:
                logger.debug('[+] %s : %s' % (key, ret_dict[key]))
            stored_filename = ret_dict['Remote file_id']
            stored_filename_without_group = stored_filename[stored_filename.index('/')+1:]
            if not filename:
                filename =stored_filename_without_group
            vmp = {'group':ret_dict['Group name'],'file_id':stored_filename_without_group,'size':ret_dict['Uploaded size'],'upload_time':int(time.time()*1000)}
            try:
                for i in self.fdfs_redis:
                    if not i.hmset(filename,vmp):
                        raise RedisError('Save Failure')
                    logger.info('Store file(%s) by buffer successful' % filename)
            except Exception,e:
                logger.error('Save info to Redis failure. rollback...')
                try:
                    ret_dict = self.fdfs_client.delete_file(stored_filename)
                except Exception,e:
                    logger.error('Error occurred while deleting: %s'%e.message)
                return None
            return vmp
     
        def remove(self,filename):
            '''
            删除文件,
            filename是用户自定义文件名
            return True|False
            '''
            fileinfo = random.choice(self.fdfs_redis).hgetall(filename)
            stored_filename = '%s/%s'%(fileinfo['group'],fileinfo['file_id'])
            try:
                ret_dict = self.fdfs_client.delete_file(stored_filename)
                logger.info('Remove stored file successful')
            except Exception,e:
                logger.error('Error occurred while deleting: %s'%e.message)
                return False
            for i in self.fdfs_redis:
                if not i.delete(filename):
                    logger.error('Remove fileinfo in redis failure')
            logger.info('%s removed.'%filename)
            return True
     
        def download(self,filename):
            '''
            下载文件
            返回二进制
            '''
            finfo = self.getInfo(filename)
            if finfo:
                ret = self.fdfs_client.download_to_buffer('%s/%s'%(finfo['group'],finfo['file_id']))
                return ret['Content']
            else:
                logger.debug('%s is not exists'%filename)
                return None
     
        def list(self,pattern='*'):
            '''
            列出文件列表
            '''
            return random.choice(self.fdfs_redis).keys(pattern)
     
        def getInfo(self,filename):
            '''
            获得文件信息
            return:{
            'group':组名,
            'file_id':不含组名的文件ID,
            'size':文件尺寸,
            'upload_time':上传时间
            }
            '''
            return random.choice(self.fdfs_redis).hgetall(filename)

  

配置：

    
    
    # -*- coding: utf-8 -*-
    #fastdfs tracker, multiple tracker supported
    fdfs_tracker = {
    'host_tuple':('192.168.2.233','192.168.2.234'),
    'port':22122,
    'timeout':30,
    'name':'Tracker Pool'
    }
    #fastdfs meta db, multiple redisdb supported
    fdfs_redis_dbs = (
        ('192.168.2.233',6379,0),
        ('192.168.2.233',6379,1)
    )

  

