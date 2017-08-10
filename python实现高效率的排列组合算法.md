# python实现高效率的排列组合算法

组合算法

 本程序的思路是开一个数组，其下标表示1到m个数，数组元素的值为1表示其下标

 代表的数被选中，为0则没选中。

 首先初始化，将数组前n个元素置1，表示第一个组合为前n个数。

 然后从左到右扫描数组元素值的“10”组合，找到第一个“10”组合后将其变为

 “01”组合，同时将其左边的所有“1”全部移动到数组的最左端。

 当第一个“1”移动到数组的m-n的位置，即n个“1”全部移动到最右端时，就得

 到了最后一个组合。

 例如求5中选3的组合：

 1   1   1   0   0   //1,2,3

 1   1   0   1   0   //1,2,4

 1   0   1   1   0   //1,3,4

 0   1   1   1   0   //2,3,4

 1   1   0   0   1   //1,2,5

 1   0   1   0   1   //1,3,5

 0   1   1   0   1   //2,3,5

 1   0   0   1   1   //1,4,5

 0   1   0   1   1   //2,4,5

 0   0   1   1   1   //3,4,5

  

使用python实现：

    
    
    group = [1, 1, 1, 0, 0, 0]
    group_len = len(group)
    #计算次数
    ret = [group]
    ret_num = (group_len * (group_len - 1) * (group_len - 2)) / 6
    for i in xrange(ret_num - 1):
     
        '第一步：先把10换成01'
        number1_loc = group.index(1)
        number0_loc = group.index(0)
     
        #替换位置从第一个０的位置开始
        location = number0_loc
          #判断第一个０和第一个１的位置哪个在前，
          #如果第一个０的位置小于第一个１的位置，
          #那么替换位置从第一个１位置后面找起
     
        if number0_loc < number1_loc:
            location = group[number1_loc:].index(0) + number1_loc
     
        group[location] = 1
        group[location - 1] = 0
     
        '第二步：把第一个10前面的所有１放在数组的最左边'
        if location - 3 >= 0:
            if group[location - 3] == 1 and group[location - 2] == 1:
                group[location - 3] = 0
                group[location - 2] = 0
                group[0] = 1
                group[1] = 1
            elif group[location - 3] == 1:
                group[location - 3] = 0
                group[0] = 1
            elif group[location - 2] == 1:
                group[location - 2] = 0
                group[0] = 1
     
        print group
        ret.append(group)

  

全排列算法

从1到N，输出全排列，共N！条。

分析：用N进制的方法吧。设一个N个单元的数组，对第一个单元做加一操作，满N进

一。每加一次一就判断一下各位数组单元有无重复，有则再转回去做加一操作，没

有则说明得到了一个排列方案。

  

