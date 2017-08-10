# 用Python写一个简单的中文分词器

解压后取出以下文件：

训练数据：icwb2-data/training/pku_ training.utf8

测试数据：icwb2-data/testing/pku_ test.utf8

正确分词结果：icwb2-data/gold/pku_ test_ gold.utf8

评分工具：icwb2-data/script/socre

2 算法描述

算法是最简单的正向最大匹配(FMM)：

用训练数据生成一个字典

对测试数据从左到右扫描，遇到一个最长的词，就切分下来，直到句子结束

注：这是最初的算法，这样做代码可以控制在60行内，后来看测试结果发现没有很好地处理数字问题， 才又增加了对数字的处理。

3 源代码及注释

    
    
    #! /usr/bin/env python
    # -*- coding: utf-8 -*-
      
    # Author: minix
    # Date:   2013-03-20
    
       
    import codecs
    import sys
       
    # 由规则处理的一些特殊符号
    numMath = [u'0', u'1', u'2', u'3', u'4', u'5', u'6', u'7', u'8', u'9']
    numMath_suffix = [u'.', u'%', u'亿', u'万', u'千', u'百', u'十', u'个']
    numCn = [u'一', u'二', u'三', u'四', u'五', u'六', u'七', u'八', u'九', u'〇', u'零']
    numCn_suffix_date = [u'年', u'月', u'日']
    numCn_suffix_unit = [u'亿', u'万', u'千', u'百', u'十', u'个']
    special_char = [u'(', u')']
       
       
    def proc_num_math(line, start):
        """ 处理句子中出现的数学符号 """
        oldstart = start
        while line[start] in numMath or line[start] in numMath_suffix:
            start = start + 1
        if line[start] in numCn_suffix_date:
            start = start + 1
        return start - oldstart
       
    def proc_num_cn(line, start):
        """ 处理句子中出现的中文数字 """
        oldstart = start
        while line[start] in numCn or line[start] in numCn_suffix_unit:
            start = start + 1
        if line[start] in numCn_suffix_date:
            start = start + 1
        return start - oldstart
       
    def rules(line, start):
        """ 处理特殊规则 """
        if line[start] in numMath:
            return proc_num_math(line, start)
        elif line[start] in numCn:
            return proc_num_cn(line, start)
       
    def genDict(path):
        """ 获取词典 """
        f = codecs.open(path,'r','utf-8')
        contents = f.read()
        contents = contents.replace(u'\r', u'')
        contents = contents.replace(u'\n', u'')
        # 将文件内容按空格分开
        mydict = contents.split(u' ')
        # 去除词典List中的重复
        newdict = list(set(mydict))
        newdict.remove(u'')
       
        # 建立词典
        # key为词首字，value为以此字开始的词构成的List
        truedict = {}
        for item in newdict:
            if len(item)>0 and item[0] in truedict:
                value = truedict[item[0]]
                value.append(item)
                truedict[item[0]] = value
            else:
                truedict[item[0]] = [item]
        return truedict
       
    def print_unicode_list(uni_list):
        for item in uni_list:
            print item,
       
    def divideWords(mydict, sentence):
        """ 
        根据词典对句子进行分词,
        使用正向匹配的算法，从左到右扫描，遇到最长的词，
        就将它切下来，直到句子被分割完闭
        """
        ruleChar = []
        ruleChar.extend(numCn)
        ruleChar.extend(numMath)
        result = []
        start = 0
        senlen = len(sentence)
        while start < senlen:
            curword = sentence[start]
            maxlen = 1
            # 首先查看是否可以匹配特殊规则
            if curword in numCn or curword in numMath:
                maxlen = rules(sentence, start)
            # 寻找以当前字开头的最长词
            if curword in mydict:
                words = mydict[curword]
                for item in words:
                    itemlen = len(item)
                    if sentence[start:start+itemlen] == item and itemlen > maxlen:
                        maxlen = itemlen
            result.append(sentence[start:start+maxlen])
            start = start + maxlen
        return result
       
    def main():
        args = sys.argv[1:]
        if len(args) < 3:
            print 'Usage: python dw.py dict_path test_path result_path'
            exit(-1)
        dict_path = args[0]
        test_path = args[1]
        result_path = args[2]
       
        dicts = genDict(dict_path)
        fr = codecs.open(test_path,'r','utf-8')
        test = fr.read()
        result = divideWords(dicts,test)
        fr.close()
        fw = codecs.open(result_path,'w','utf-8')
        for item in result:
            fw.write(item + ' ')
        fw.close()
       
    if __name__ == "__main__":
        main()

4 测试及评分结果  

使用 dw.py 训练数据 测试数据， 生成结果文件

使用 score 根据训练数据，正确分词结果，和我们生成的结果进行评分

使用 tail 查看结果文件最后几行的总体评分，另外socre.utf8中还提供了大量的比较结果， 可以用于发现自己的分词结果在哪儿做的不够好

注：整个测试过程都在Ubuntu下完成

$ python dw.py pku_training.utf8 pku_test.utf8 pku_result.utf8

$ perl score pku_training.utf8 pku_test_gold.utf8 pku_result.utf8 > score.utf8

$ tail -22 score.utf8

INSERTIONS:     0

DELETIONS:      0

SUBSTITUTIONS:  0

NCHANGE:        0

NTRUTH: 27

NTEST:  27

TRUE WORDS RECALL:      1.000

TEST WORDS PRECISION:   1.000

=== SUMMARY:

=== TOTAL INSERTIONS:   4623

=== TOTAL DELETIONS:    1740

=== TOTAL SUBSTITUTIONS:        6650

=== TOTAL NCHANGE:      13013

=== TOTAL TRUE WORD COUNT:      104372

=== TOTAL TEST WORD COUNT:      107255

=== TOTAL TRUE WORDS RECALL:    0.920

=== TOTAL TEST WORDS PRECISION: 0.895

=== F MEASURE:  0.907

=== OOV Rate:   0.940

=== OOV Recall Rate:    0.917

=== IV Recall Rate:     0.966

  

基于词典的FMM算法是非常基础的分词算法，效果没那么好，不过足够简单，也易于入手，随着学习的深入，我可能还会用Python实现其它的分词算法。另外一个感受是
，看书的时候尽量多去实现，这样会让你有足够的热情去关注理论的每一个细节，不会感到那么枯燥无力。

  

