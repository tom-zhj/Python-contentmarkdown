# Python将阿拉伯数字转化为中文大写

利用Python将阿拉伯数字转化为中文大写，其实最麻烦的地方就是中间空多个0的问题，这种情况下，采用拆分法则，将一个大数字，先拆分成整数部分和小数部分，再对
整数部分按照仟、万、亿、兆分位拆分为四个字符串组成的List，每个字符串最多4个字符，然后对每个分位的字符串用大写函数转换成大写，最后合并，这样等于缩减了问
题，处理就相对简单了  

    
    
    #!/usr/bin/env python
    # -*- coding: utf-8 -*-
    '''
    #算法说明：要求字符串输入，现将字符串差费为整数部分和小数部分生成list[整数部分,小数部分]
    #将整数部分拆分为：[亿，万，仟]三组字符串组成的List:['0000','0000','0000']（根据实际输入生成阶梯List）
    #例如：600190000010.70整数部分拆分为：['600','1900','0010']
    #然后对list中每个字符串分组进行大写化再合并
    #最后处理小数部分的大写化
    '''
    class cnumber:
        cdict={}
        gdict={}
        xdict={}
        def __init__(self):
            self.cdict={1:u'',2:u'拾',3:u'佰',4:u'仟'}
            self.xdict={1:u'元',2:u'万',3:u'亿',4:u'兆'} #数字标识符
            self.gdict={0:u'零',1:u'壹',2:u'贰',3:u'叁',4:u'肆',5:u'伍',6:u'陆',7:u'柒',8:u'捌',9:u'玖'}     
        def csplit(self,cdata): #拆分函数，将整数字符串拆分成[亿，万，仟]的list
            g=len(cdata)%4
            csdata=[]
            lx=len(cdata)-1
            if g>0:
                csdata.append(cdata[0:g])
            k=g
            while k<=lx:
                csdata.append(cdata[k:k+4])
                k+=4
            return csdata
                     
        def cschange(self,cki): #对[亿，万，仟]的list中每个字符串分组进行大写化再合并
            lenki=len(cki)
            i=0
            lk=lenki
            chk=u''
            for i in range(lenki):
                if int(cki[i])==0:
                    if i

