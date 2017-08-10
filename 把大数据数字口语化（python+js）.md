# 把大数据数字口语化（python+js）

当出现万以上的整型数字时，经常要把它们口语化比较直观。下面分享两段代码，python与js的

python

    
    
    def fn(num):
        '''
        把数字口语化
        '''
         
        ret = ''
        num = int(num)
        if num/10000 == 0:
            ret = str(num)
        else:
            if num/10**8 == 0:
                if num%10000 != 0:
                    ret = str(num/10000) + '万' + str(num % 10000)
                else:
                    ret = str(num/10000) + '万'
            else:
                n2 = num%10**8
                if n2%10000 != 0 and n2/10000 != 0:
                    ret = str(num/10**8) + '亿' + str(n2/10000) + '万' + str(n2%10000)
                elif  n2%10000 != 0 and n2/10000 == 0:
                    ret = str(num/10**8) + '亿' +  str(n2%10000)
                elif  n2%10000 == 0 and n2/10000 != 0:
                    ret = str(num/10**8) + '亿' +  str(n2/10000) + '万'
                elif  n2%10000 == 0 and n2/10000 == 0:
                    ret = str(num/10**8) + '亿'
        return ret

  

javascript:

    
    
    function int2string(num) {
        num = Number(num);
        if (num/10000 < 1){
            ret = num;
        }else{
            if (num/Math.pow(10,8) < 1) {
                if (num%10000 != 0) {
                    ret = parseInt(num/10000) + '万' + num % 10000;
                }else{
                    ret = parseInt(num/10000) + '万';
                }
            }else{
                n2 = num%Math.pow(10,8);
                if (n2%10000 != 0 & n2/10000 != 0) {
                    ret = parseInt(num/Math.pow(10,8)) + '亿' + parseInt(n2/10000) + '万' + (n2%10000);
                }else if(n2%10000 != 0 & n2/10000 == 0){
                    ret = parseInt(num/Math.pow(10,8)) + '亿' +  parseInt(n2%10000);
                }else if(n2%10000 == 0 & n2/10000 != 0){
                    ret = parseInt(num/Math.pow(10,8)) + '亿' +  parseInt(n2/10000) + '万';
                }else if(n2%10000 == 0 & n2/10000 == 0){
                    ret = (num/Math.pow(10,8)) + '亿';
                }
            }
        }
        return ret
    }

  

  

