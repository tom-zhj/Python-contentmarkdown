# python读取和生成excel文件

今天来看一下如何使用python处理excel文件，处理excel文件是在工作中经常用到的，python为我们考虑到了这一点，python中本身就自带csv
模块。

**1.用python读取csv文件**：

csv是逗号分隔符格式 一般我们用的execl生成的格式是xls和xlsx  直接重命名为csv的话会报错：

Error: line contains NULL byte

insun解决方案：出错原因是直接是把后缀为xls的execl文件重命名为csv的 正常的要是另存为csv文件 就不会报错了

譬如我们有这么个csv文件：

![](http://www.pythontab.com/uploadfile/2013/0314/20130314102231751.jpg)  

    
    
    #!/usr/bin/env python
    # -*- coding:utf-8 -*-
     
    import csv
    with open('egg.csv','rb') as f:
    reader = csv.reader(f)
    for row in reader:
    print row

  

打印出来是这样的list  

['a', '1', '1', '1']

['a', '2', '2', '2']

['b', '3', '3', '3']

['b', '4', '4', '4']

['b', '5', '5', '5']

['b', '6', '6', '6']

['c', '7', '7', '7']

['c', '8', '8', '8']

['c', '9', '9', '9']

['c', '10', '10', '10']

['d', '11', '11', '11']

['e', '12', '12', '12']

['e', '13', '13', '13']

['e', '14', '14', '14']

**2.用python写入并生成csv**
    
    
    #!/usr/bin/env python
    # -*- coding:utf-8 -*-
     
    import csv
    with open('egg2.csv', 'wb') as csvfile:
    spamwriter = csv.writer(csvfile, delimiter=' ',quotechar='|', quoting=csv.QUOTE_MINIMAL)
    spamwriter.writerow(['a', '1', '1', '2', '2'])
    spamwriter.writerow(['b', '3', '3', '6', '4'])
    spamwriter.writerow(['c', '7', '7', '10', '4'])
    spamwriter.writerow(['d', '11','11','11', '1'])
    spamwriter.writerow(['e', '12','12','14', '3'])

![](http://www.pythontab.com/uploadfile/2013/0314/20130314102236328.jpg)  

这样存进去的是存到一列了 跟我们原本意图存进5列不一样

使用python的csv生成excel所兼容的csv文件的话，主要就是创建writer时的参数时要有dialect=’excel’

代码修改为：

    
    
    #!/usr/bin/env python
    # -*- coding:utf-8 -*-
     
    import csv
    with open('egg2.csv', 'wb') as csvfile:
    spamwriter = csv.writer(csvfile,dialect='excel')
    spamwriter.writerow(['a', '1', '1', '2', '2'])
    spamwriter.writerow(['b', '3', '3', '6', '4'])
    spamwriter.writerow(['c', '7', '7', '10', '4'])
    spamwriter.writerow(['d', '11','11','11', '1'])
    spamwriter.writerow(['e', '12','12','14', '3'])

  

