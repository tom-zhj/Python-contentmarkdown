# python时间处理详解

1.获取当前时间的两种方法：

  

import datetime,time

now = time.strftime("%Y-%m-%d %H:%M:%S")

print now

now = datetime.datetime.now()

print now

2.获取上个月最后一天的日期(本月的第一天减去1天)

  

last = datetime.date(datetime.date.today().year,datetime.date.today().month,1)
-datetime.timedelta(1)

print last

3.获取时间差(时间差单位为秒，常用于计算程序运行的时间)

  

starttime = datetime.datetime.now()

#long running

endtime = datetime.datetime.now()

print (endtime - starttime).seconds

  

4.计算当前时间向后10个小时的时间



d1 = datetime.datetime.now()

d3 = d1 + datetime.timedelta(hours=10)

d3.ctime()

  

其本上常用的类有：datetime和timedelta两个。它们之间可以相互加减。每个类都有一些方法和属性可以查看具体的值，如
datetime可以查看：天数(day)，小时数(hour)，星期几(weekday())等;timedelta可以查看：天数(days)，秒数
(seconds)等。

  

5.python中时间日期格式化符号：

  

%y 两位数的年份表示（00-99）

%Y 四位数的年份表示（000-9999）

%m 月份（01-12）

%d 月内中的一天（0-31）

%H 24小时制小时数（0-23）

%I 12小时制小时数（01-12）

%M 分钟数（00=59）

%S 秒（00-59）

  

%a 本地简化星期名称

%A 本地完整星期名称

%b 本地简化的月份名称

%B 本地完整的月份名称

%c 本地相应的日期表示和时间表示

%j 年内的一天（001-366）

%p 本地A.M.或P.M.的等价符

%U 一年中的星期数（00-53）星期天为星期的开始

%w 星期（0-6），星期天为星期的开始

%W 一年中的星期数（00-53）星期一为星期的开始

%x 本地相应的日期表示

%X 本地相应的时间表示

%Z 当前时区的名称

%% %号本身

附上示例代码：

代码Code highlighting produced by Actipro CodeHighlighter
(freeware)http://www.CodeHighlighter.com/-->#-*-coding:utf-8-*-

import datetime, calendar



def getYesterday():

    today=datetime.date.today()

    oneday=datetime.timedelta(days=1)

    yesterday=today-oneday

   return yesterday



def getToday():

    return datetime.date.today()



#获取给定参数的前几天的日期，返回一个list

def getDaysByNum(num):

     today=datetime.date.today()

     oneday=datetime.timedelta(days=1)

     li=[]

    for i in range(0,num):

        #今天减一天，一天一天减

         today=today-oneday

        #把日期转换成字符串

        #result=datetostr(today)

         li.append(datetostr(today))

    return li



#将字符串转换成datetime类型

def strtodatetime(datestr,format):

    return datetime.datetime.strptime(datestr,format)



#时间转换成字符串,格式为2008-08-02

def datetostr(date):

    return    str(date)[0:10]



#两个日期相隔多少天，例：2008-10-03和2008-10-01是相隔两天

def datediff(beginDate,endDate):

     format="%Y-%m-%d";

     bd=strtodatetime(beginDate,format)

     ed=strtodatetime(endDate,format)

     oneday=datetime.timedelta(days=1)

     count=0

    while bd!=ed:

         ed=ed-oneday

         count+=1

    return count



#获取两个时间段的所有时间,返回list

def getDays(beginDate,endDate):

     format="%Y-%m-%d";

     bd=strtodatetime(beginDate,format)

     ed=strtodatetime(endDate,format)

     oneday=datetime.timedelta(days=1)

     num=datediff(beginDate,endDate)+1

     li=[]

    for i in range(0,num):

         li.append(datetostr(ed))

         ed=ed-oneday

    return li



#获取当前年份 是一个字符串

def getYear():

    return str(datetime.date.today())[0:4]



#获取当前月份 是一个字符串

def getMonth():

    return str(datetime.date.today())[5:7]



#获取当前天 是一个字符串

def getDay():

    return str(datetime.date.today())[8:10]

def getNow():

    return datetime.datetime.now()





print getToday()

print getYesterday()

print getDaysByNum(3)

print getDays('2008-10-01','2008-10-05')

print '2008-10-04 00:00:00'[0:10]



print str(getYear())+getMonth()+getDay()

print getNow()

  

