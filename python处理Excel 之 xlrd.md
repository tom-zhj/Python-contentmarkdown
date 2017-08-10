# python处理Excel 之 xlrd

python处理Excel常用到的模块是xlrd。使用xlrd可以非常方便的处理Excel文档，下面介绍一下基本用法

1.打开文件

import xlrd

data= xlrd.open_workbook("c:\\\skills.xls")

获取一个工作表

table = data.sheet_by_name(u'skills') #也可以

table = data.sheet_by_index(0)

行，列的获取

table.row_values(i)

table.col_values(i)

行数，列数 等

nrows = table.nrows

ncols = table.ncols

单元格数据

cell_A1 = table.cell(0, 0).value

cell_C4 = table.cell(2, 3).value

#简单写单元格

table.put_cell(row, col, ctype, value, xf)

row = col = 0

ctype = 1 # 0 empty , 1 string, 2 number, 3 date, 4 bool, 5 error

value = 'this is cell value'

xf = 0

  

更多功能请参照官方文档

<https://secure.simplistix.co.uk/svn/xlrd/trunk/xlrd/doc/xlrd.html?p=4966>

  

