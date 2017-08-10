#  用Python备份MYSQL 数据库

 工作需要，对公司的MYSQL数据库进行备份，赶上刚刚开始学python，看了一套简单的python教学视频，简单的写了个备份脚本，个人表示 对python
的class 、function、build-in function 、私有变量、全局变量 等等，该怎么用，啥时候用等 毫无概念
，仅此记录一下吧，也欢迎路过的pythoner赐教。

个人已知的一些问题：

   1、该脚本必须要求 mysql配置文件内的所有行为 key=value的格式，并且不能存在多余的注释，否则ConfigParser模块解析配置文件时会
出错，由于没研究过ConfigParser是不是有容错的方法可以调用，也没时间写容错处理，而是通过整理my.ini
配置文件使其符合ConfigParser的要求解决的。后面会附上我用的mysql配置文件。

   2、大量使用类私有成员变量，因为完全不知道python 变量、类方法、等等啥时候该私有化，以及有啥区别，只知道类私有成员变量在别的脚本中import
或者继承时，是不可见的。

   3、比较多的进行文件操作，以及传值操作，目前只保证按正确格式传值没问题，没有做多余的容错处理。    4、大量的在进行字符串拼接，第一次写运维相关脚本
，由于要调用系统命令，和传递很多参数，也不会subprocess模块，不知道别人写运维脚本都具体咋做，就直接拼接了。

   5、其他未知的bug、未发现的逻辑错误等等。

环境：

\- Server :             Dell PowerEdge T110

\- OS:                   CentOS 6.3_x86_64

\- PythonVersion:    2.7.3

\- MysqlVersion:      5.5.28 linux x86_64

### MysqlBackupScript.py

    
    
    #!/usr/bin/env python
    # coding: utf8
    # script MysqlBackupScript
    # by Becareful
    # version v1.0
     
    """
    This scripts provides auto backup mysql(version == 5.5.x) database .
    """
     
    import os
    import sys
    import datetime         #用于生成备份文件的日期
    import linecache        #用于读取文件的指定行
    import ConfigParser     #解析mysql配置文件
     
     
    class DatabaseArgs(object):
        """
     
        """
     
        __MYSQL_BASE_DIR = r'/usr/local/mysql'            #mysql安装目录
        __MYSQL_BIN_DIR = __MYSQL_BASE_DIR + '/bin'       #mysql二进制目录
        __MYSQL_CONFIG_FILE = r'/usr/local/mysql/my.cnf'  #mysql配置文件
     
        __ONEDAY = datetime.timedelta(days=1)             #一天的时长，用于计算下面的前一天和后一天日期
        __TODAY = datetime.date.today()      #当天日期格式为 YYYY-MM-DD
        __YESTERDAY = __TODAY - __ONEDAY                  #计算昨天日期
        __TOMORROW = __TODAY + __ONEDAY                   #计算明天日期
        __WEEKDAY = __TODAY.strftime('%w')                #计算当天是一星期的星期几
     
        __MYSQL_DUMP_ARGS = {                             #用一个字典存储mysqldump 命令备份数据库的参数
            'MYISAM': ' -v -E -e -R --triggers -F  -n --opt --master-data=2 --hex-blob -B ',
            'INNODB': ' -v -E -e -R --triggers -F --single-transaction -n --opt --master-data=2 --hex-blob -B '
        }
     
        __DUMP_COMMAND = __MYSQL_BIN_DIR + '/mysqldump'   #mysqldump 命令的 路径 用于dump mysql数据
        __FLUSH_LOG_COMMAND = __MYSQL_BIN_DIR + '/mysqladmin'    #mysqladmin 命令的路径 ，用于执行 flush-logs 生成每天增量binlog
     
        __BACKUP_DIR = r'/backup/'                        # 指定备份文件存放的目录
     
        __PROJECTNAME = 'example'                         # 指定需要备份的数据库对应的项目名，将来会生成 projectname-YYYY-MM-DD.sql 等文件
        __DATABASE_LIST = []                              # 指定需要备份的数据库名，可以是多个，使用列表
        __HOST = 'localhost'
        __PORT = 3306
        __USERNAME = 'root'
        __PASSWORD = ''
        __LOGINARGS = ''                                  # 如果在localhost登陆，需要密码，可以设定登陆的参数，具体在下面有说明
        __LOGFILE = __BACKUP_DIR + '/backup.logs'
     
        def __init__(self, baseDir=__MYSQL_BASE_DIR, backDir=__BACKUP_DIR, engine='MYISAM', projectName=__PROJECTNAME,
                     dbList=__DATABASE_LIST, host=__HOST, port=__PORT, user=__USERNAME, passwd=__PASSWORD):
     
            """
                实例化对象时传入的参数，如不传入默认使用类的私有成员变量作为默认值
        :param baseDir:    
        :param backDir:
        :param engine:
        :param projectName:
        :param dbList:
        :param host:
        :param port:
        :param user:
        :param passwd:
        """
            self.__MYSQL_BASE_DIR = baseDir
            self.__BACKUP_DIR = backDir
            self.__PROJECTNAME = projectName
            self.__DATABASE_LIST = dbList
            self.__HOST = host
            self.__PORT = port
            self.__USERNAME = user
            self.__PASSWORD = passwd
            self.__ENGINE = self.__MYSQL_DUMP_ARGS[engine]
            #下面定义了如需登陆时，参数 其实就是生成 这样的格式  “-hlocalhost -uroot --password=‘xxxx’”
            self.__LOGINARGS = " -h" + self.__HOST + " -P" + str(
                self.__PORT) + " -u" + self.__USERNAME + " --password='" + self.__PASSWORD + "'"
            self.checkDatabaseArgs()   #调用检查函数
     
        def __getconfig(self, cnf=__MYSQL_CONFIG_FILE, item=None):  # 解析mysql配置文件的小函数，简单封装了下，传入一个值作为my.cnf的key去查找对应的value
     
            __mycnf = ConfigParser.ConfigParser()
            __mycnf.read(cnf)
            try:
                return __mycnf.get("mysqld", item)
            except BaseException, e:
                sys.stderr.write(str(e))
                sys.exit(1)
     
        def __getBinlogPath(self): #  取每天需要增量备份的binlog日志的绝对路径，从mysql的binlog.index文件取倒数第二行
     
            __BINLOG_INDEX = self.__getconfig(item='log-bin') + '.index'
     
            if not os.path.isfile(__BINLOG_INDEX):
                sys.stderr.write('BINLOG INDEX FILE: [' + __BINLOG_INDEX + ' ] NOT FOUND! \n')
                sys.exit(1)
            else:
                try:
                    __BINLOG_PATH = linecache.getline(__BINLOG_INDEX, len(open(__BINLOG_INDEX, 'r').readlines()) - 1)
                    linecache.clearcache()
                except BaseException, e:
                    sys.stderr.write(str(e))
                    sys.exit(1)
                return __BINLOG_PATH.strip()
     
        def flushDatabaseBinlog(self):  # 调用此函数，将会执行  mysqladmin flush-logs ,刷新binlog日志
            return os.popen(self.__FLUSH_LOG_COMMAND + self.__LOGINARGS + ' flush-logs')
     
        def dumpDatabaseSQL(self):  #|通过mysqladmin 对指定数据库进行全备
            if not os.path.isfile(self.__BACKUP_DIR + '/' + self.__PROJECTNAME + '/' +  str(self.__YESTERDAY) + '-' + self.__PROJECTNAME + '.sql'):
                return os.popen(self.__DUMP_COMMAND + self.__LOGINARGS + self.__ENGINE + ' '.join(
                    self.__DATABASE_LIST) + ' >> ' + self.__BACKUP_DIR + '/' + self.__PROJECTNAME + '/' + str(
                    self.__YESTERDAY) + '-' + self.__PROJECTNAME + '.sql')
            else:
                sys.stderr.write('Backup File [' + str(self.__YESTERDAY) + '-' + self.__PROJECTNAME + '.sql]  already exists.\n')
     
     
     
        def dumpDatabaseBinlog(self):#通过copy2() 将需要备份的binlog日志复制到指定备份目录
     
            if not os.path.isfile(self.__BACKUP_DIR + '/' + self.__PROJECTNAME + '/' + str(self.__YESTERDAY) + '-' + os.path.split(self.__getBinlogPath())[1]):
                from shutil import copy2
                try:
                    copy2(self.__getBinlogPath(), self.__BACKUP_DIR + '/' + self.__PROJECTNAME + '/' + str(self.__YESTERDAY) + '-' + os.path.split(self.__getBinlogPath())[1])
                except BaseException, e:
                    sys.stderr.write(str(e))
            else:
                sys.stderr.write('Binlog File [' + str(self.__YESTERDAY) + '-' + os.path.split(self.__getBinlogPath())[1] + '] already exists\n' )
     
     
        def checkDatabaseArgs(self):  #对一些必要条件进行检查
            __rv = 0
     
            if not os.path.isdir(self.__MYSQL_BASE_DIR):  #检查指定的mysql安装目录是否存在
                sys.stderr.write('MYSQL BASE DIR: [ ' + self.__MYSQL_BASE_DIR + ' ] NOT FOUND\n')
                __rv += 1
     
            if not os.path.isdir(self.__BACKUP_DIR):   #检查指定的备份目录是否存在，如不存在自动创建
                sys.stderr.write('BACKUP DIR: [ ' + self.__BACKUP_DIR + '/' + self.__PROJECTNAME +  ' ] NOT FOUND ,AUTO CREATED\n')
                os.makedirs(self.__BACKUP_DIR + '/' + self.__PROJECTNAME)
     
            if not os.path.isfile(self.__MYSQL_CONFIG_FILE): #检查mysql配置文件是否存在
                sys.stderr.write('MYSQL CONFIG FILE: [' + self.__MYSQL_CONFIG_FILE + ' ] NOT FOUND\n')
                __rv += 1
     
            if not os.path.isfile(self.__DUMP_COMMAND):  #检查备份数据库时使用的mysqldump命令是否存在
                sys.stderr.write('MYSQL DUMP COMMAND: [' + self.__DUMP_COMMAND + ' ] NOT FOUND\n')
                __rv += 1
     
            if not os.path.isfile(self.__FLUSH_LOG_COMMAND): #检查刷新mysql binlog日志使用的mysqladmin命令是否存在
                sys.stderr.write('MYSQL FLUSH LOG COMMAND: [' + self.__DUMP_COMMAND + ' ] NOT FOUND\n')
                __rv += 1
     
            if not self.__DATABASE_LIST:  #检查需要备份的数据库列表是否存在
                sys.stderr.write('Database List is None \n')
                __rv += 1
     
            if __rv:   # 判断返回值，由于上述任何一步检查失败，都会导致 __rv 值 +1 ，只要最后__rv ！= 0就直接退出了。
                sys.exit(1)
     
     
    def crontab():  # 使用字典，来进行相关参数传递，并实例化对象，调用相关方法进行操作
     
        zabbix = {
            'baseDir': '/usr/local/mysql/',
            'backDir': '/backup/',
            'projectName': 'Monitor',
            'dbList': ['zabbix'],
            'host': 'localhost',
            'port': 3306,
            'user': 'root',
            'passwd': 'xxxxxxx'
        }
     
        monitor = DatabaseArgs(**zabbix)
        monitor.dumpDatabaseSQL()
        monitor.dumpDatabaseBinlog()
        monitor.flushDatabaseBinlog()
     
    if __name__ == '__main__':
        crontab()

### my.cnf

    
    
    [client]
    port                            = 3306
    socket                          = /mysql/var/db.socket
     
    [mysqld]
    socket                          = /mysql/var/db.socket
    datadir                         = /mysql/db/
    skip-external-locking           = 1
    skip-innodb                     = 0 
    key_buffer_size                 = 256M
    max_allowed_packet              = 10M
    table_open_cache                = 2048
    sort_buffer_size                = 4M
    read_buffer_size                = 4M
    read_rnd_buffer_size            = 8M
    myisam_sort_buffer_size         = 64M
    myisam_max_sort_file_size       = 1G
    myisam_repair_threads           = 1
    myisam_recover                  = DEFAULT
    thread_cache_size               = 32
    query_cache_size                = 32M
    query_cache_min_res_unit        = 2k
    bulk_insert_buffer_size         = 64M
    tmp_table_size                  = 128M
    thread_stack                    = 192K
    skip-name-resolve               = 1
    max_connections                 = 65500
    default-storage-engine          = myisam
    federated                       = 0
    server-id                       = 1
    slave-skip-errors               = all
    #log                            = /var/log/sql_query.log
    slow-query-log                  = 1
    slow-query-log-file             = /mysql/log/sql_query_slow.log
    long-query-time                 = 5
    log-queries-not-using-indexes   = 1
    log-slow-admin-statements       = 1
    log-bin                         = /mysql/var/log/binlog/bin-log
    log-error                       = /mysql/var/log/mysql.err
    master-info-file                = /mysql/var/log/master.info
    relay-log                       = /mysql/var/log/relay-bin/relay-bin
    relay-log-index                 = /mysql/var/log/relay-bin/relay-bin.index
    relay-log-info-file             = /mysql/var/log/relay-bin/relay-bin.info
    binlog_cache_size               = 8M
    binlog_format                   = MIXED
    max_binlog_cache_size           = 20M
    max_binlog_size                 = 1G
    binlog-ignore-db                = mysql
    binlog-ignore-db                = performance_schema
    binlog-ignore-db                = information_schema
    replicate-ignore-db             = mysql
    replicate-ignore-db             = performance_schema
    replicate-ignore-db             = information_schema
     
    innodb_data_home_dir            = /mysql/ibdata/
    innodb_data_file_path           = ibdata:156M:autoextend
    innodb_log_group_home_dir       = /mysql/ibdata/
    log-slave-updates               = 0
    back_log                        = 512
    transaction_isolation           = READ-COMMITTED
    max_heap_table_size             = 246M
    interactive_timeout             = 120
    wait_timeout                    = 120
    innodb_additional_mem_pool_size = 16M
    innodb_buffer_pool_size         = 512M
    innodb_file_io_threads          = 4
    innodb_thread_concurrency       = 8
    innodb_flush_log_at_trx_commit  = 2
    innodb_log_buffer_size          = 16M
    innodb_log_file_size            = 128M
    innodb_log_files_in_group       = 3
    innodb_max_dirty_pages_pct      = 90
    innodb_lock_wait_timeout        = 120
    innodb_file_per_table           = 1
    innodb_open_file                = 327500
    open_files_limit                = 327500
     
    [mysqldump]
    quick                           = 1
    max_allowed_packet              = 50M
     
    [mysql]
    auto-rehash                     = 1
    socket                          = /mysql/var/db.socket
    safe-updates                    = 0
     
    [myisamchk]
    key_buffer_size                 = 256M
    sort_buffer_size                = 256M
    read_buffer                     = 2M
    write_buffer                    = 2M
     
    [mysqlhotcopy]
    interactive-timeout             = 100

### 最终生成的备份目录结构是这样的

    
    
    [root@zabbix backup]# find ./
    ./
    ./Monitor
    ./Monitor/2013-03-16-bin-log.000008
    ./Monitor/2013-03-14-bin-log.000006
    ./Monitor/2013-03-16-Monitor.sql
    ./Monitor/2013-03-15-Monitor.sql
    ./Monitor/2013-03-15-bin-log.000007
    ./Monitor/2013-03-14-Monitor.sql
     
    ~END~

  

