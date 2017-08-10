# sys.argv[] 的使用详解

sys.argv[]是用来获取命令行参数的，sys.argv[0]表示代码本身文件路径;比如在CMD命令行输入 &ldquo;python  test.py
-help&rdquo;，那么sys.argv[0]就代表&ldquo;test.py&rdquo;。

sys.startswith()
是用来判断一个对象是以什么开头的，比如在python命令行输入&ldquo;'abc'.startswith('ab')&rdquo;就会返回True

以下实例参考:

    
    
    #!/usr/local/bin/env python
    import sys
    def readfile(filename):
        '''Print a file to the standard output.'''
        f = file(filename)
        while True:
              line = f.readline()
              if len(line) == 0:
                 break
              print line,
        f.close()
    print "sys.argv[0]---------",sys.argv[0]                                    
    print "sys.argv[1]---------",sys.argv[1]                                    
    print "sys.argv[2]---------",sys.argv[2]
    # Script starts from here
    if len(sys.argv) < 2:
        print 'No action specified.'
        sys.exit()
    if sys.argv[1].startswith('--'):
       option = sys.argv[1][2:]
       # fetch sys.argv[1] but without the first two characters
       if option == 'version':
          print 'Version 1.2'
       elif option == 'help':
          print '''"
               This program prints files to the standard output.
               Any number of files can be specified.
               Options include:
               --version : Prints the version number
               --help    : Display this help'''
       else:
           print 'Unknown option.'
           sys.exit()
    else:
        for filename in sys.argv[1:]:
            readfile(filename)
    执行结果:# python test.py --version help
    sys.argv[0]--------- test.py
    sys.argv[1]--------- --version
    sys.argv[2]--------- help
    Version 1.2

注意:sys.argv[1][2:]表示从第二个参数，从第三个字符开始截取到最后结尾，本例结果为:version

  

