# Python 执行js的2种解决方案

第1种方案

SpiderMonkey是Mozilla项目的一部分，是一个用C语言实现的JavaScript脚本引擎，
该引擎分析、编译和执行脚本，根据JS数据类型和对象的需要进行内存分配及释放操作；利用该引擎可以让你的应用程序具有解释JavaScript脚本的能力。

要想使用spidermonkey得先安装，方法如下：

cd /home/linuxany.com/

wget http://ftp.mozilla.org/pub/mozilla.org/js/js-1.7.0.tar.gz -O- | tar xvz

cd js/src

make -f Makefile.ref

mkdir -p /usr/include/smjs/ -v

cp *.{h,tbl} /usr/include/smjs/ -v

cd Linux_All_DBG.OBJ

cp *.h /usr/include/smjs/ -v

mkdir -p /usr/local/{bin,lib}/ -v

cp js /usr/local/bin/ -v

cp libjs.so /usr/local/lib/ -v

以上安装完成后，运行/usr/local/bin/js 就应该可以启动js解释运行引擎了.

python使用举例：

    
    
    # coding:utf-8
    import os
    import tempfile
    def call_js(js):
        f=tempfile.mktemp('sd', 'linuxany', '/tmp')
        f2=tempfile.mktemp('sd', 'linuxany', '/tmp')
                       
        fp=open(f,'w')
        fp.write(js)
        fp.close()
                       
        cmd="/usr/local/bin/js  %s > %s" % (f,f2)
                       
        os.system(cmd)
        result=open(f2).read()
        print result
    if __name__ == "__main__":
        code='''
        function dF(s,n){
            n=parseInt(n);
            var s1=unescape(s.substr(0,n)+s.substr(n+1,s.length-n-1));
            var t='';
            for(var i=0;i第2种方案Python-Spidermonkey 这个Python模块允许执行Javascript相关功能，是python与javascript之间进行操作的桥梁，javascript的类，对象和函数都可以在Python中调用。它大量借鉴了克拉斯Jacobssen的JavaScript Perl模块，而这又是Mozilla的PerlConnect Perl的结合为基础。安装：svn checkout http://python-spidermonkey.googlecode.com/svn/trunk/ python-spidermonkey-read-only下载完后，先运行python setup.py build然后运行python setup.py install官方网站：http://code.google.com/p/python-spidermonkey/同时需要安装Pyrex模块，一个支持python和C语言混编的模块。装完后就用python其他模块一样使用即可。

