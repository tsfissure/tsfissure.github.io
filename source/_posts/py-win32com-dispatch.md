---
title: py_win32com_dispatch
date: 2018-05-03 14:22:22
tags: [py,excel]
---


有个朋友叫帮忙写个脚本,把Excel A,B,C,D,E中的某个sheet，同步到F中。我觉得这很容易，便一口答应了。
但试了好几个库比如openpyxl, pandas,xlrd,xlwt啥的，都有两个共同问题
- 1. 不能把sheet复制到不同的Excel中
- 2. 复制单元格内容时，只能复制值，格式控制不完全,图表啥的，就GG了.

最后发现 win32com.client.Dispatch，异常好用，记录一下。

<!-- more -->

### 优势

相比其他库，这里优势在于跨Excel的复制，其实其他库也提供了copy方法，但都有个判断，sheet的parent(也就是Excel)是不是同一个。如果不是，就raise error.


### 亮点方法

下面代码中的Copy方法就是本文的重点了.t1中的test_sheet复制到t2中去。而且位置是在第1个的前面。
```python
from win32com.client import Dispatch

excel_dp = Dispatch("Excel.Application")
t1 = excel_dp.Workbooks.Open(Filename = "E:\\tstfxml\\ex1.xlsx") # use absolute path
t2 = excel_dp.Workbooks.Open(Filename = "E:\\tstfxml\\ex2.xlsx")

test_sheet = t1.Worksheets("test_sheet") # get sheet "test_sheet"
test_sheet.Copy(Before = t2.Worksheets(1)) # copy to ex2 before sheet_1

t2.Close(SaveChanges = True)
t1.Close()
excel_dp.Quit()
```

这里有点困扰的就是，我试过用After参数，但并不能达到满意的效果。不知道算bug还是开发者的设定。所以你想放到最后一个的话，你可能需要先添加一个临时sheet，复制过去，再删除.
