# python 探测网站目录的GUI程序

1.pyqt4写的界面 find_ui.py

    
    
    #-*- coding: utf-8 -*-
    from PyQt4 import QtCore, QtGui
     
    try:
        _fromUtf8 = QtCore.QString.fromUtf8
    except AttributeError:
        def _fromUtf8(s):
            return s
     
    try:
        _encoding = QtGui.QApplication.UnicodeUTF8
        def _translate(context, text, disambig):
            return QtGui.QApplication.translate(context, text, disambig, _encoding)
    except AttributeError:
        def _translate(context, text, disambig):
            return QtGui.QApplication.translate(context, text, disambig)
     
    class Ui_Form(object):
        def setupUi(self, Form):
            Form.setObjectName(_fromUtf8("Form"))
            Form.resize(516, 467)
            self.label = QtGui.QLabel(Form)
            self.label.setGeometry(QtCore.QRect(20, 10, 54, 16))
            self.label.setObjectName(_fromUtf8("label"))
            self.edit_address = QtGui.QLineEdit(Form)
            self.edit_address.setGeometry(QtCore.QRect(80, 10, 351, 20))
            self.edit_address.setObjectName(_fromUtf8("edit_address"))
            self.button_search = QtGui.QPushButton(Form)
            self.button_search.setGeometry(QtCore.QRect(440, 10, 61, 23))
            self.button_search.setObjectName(_fromUtf8("button_search"))
            self.text_all = QtGui.QTextEdit(Form)
            self.text_all.setGeometry(QtCore.QRect(20, 40, 411, 261))
            self.text_all.setHorizontalScrollBarPolicy(QtCore.Qt.ScrollBarAlwaysOff)
            self.text_all.setObjectName(_fromUtf8("text_all"))
            self.label_2 = QtGui.QLabel(Form)
            self.label_2.setGeometry(QtCore.QRect(20, 320, 54, 12))
            self.label_2.setObjectName(_fromUtf8("label_2"))
            self.text_exist = QtGui.QTextEdit(Form)
            self.text_exist.setGeometry(QtCore.QRect(20, 340, 411, 64))
            self.text_exist.setHorizontalScrollBarPolicy(QtCore.Qt.ScrollBarAlwaysOff)
            self.text_exist.setObjectName(_fromUtf8("text_exist"))
            self.label_3 = QtGui.QLabel(Form)
            self.label_3.setGeometry(QtCore.QRect(380, 310, 91, 20))
            self.label_3.setText(_fromUtf8(""))
            self.label_3.setObjectName(_fromUtf8("label_3"))
            self.edit_add = QtGui.QLineEdit(Form)
            self.edit_add.setGeometry(QtCore.QRect(20, 420, 411, 20))
            self.edit_add.setObjectName(_fromUtf8("edit_add"))
            self.button_add = QtGui.QPushButton(Form)
            self.button_add.setGeometry(QtCore.QRect(440, 420, 71, 23))
            self.button_add.setObjectName(_fromUtf8("button_add"))
            self.label_4 = QtGui.QLabel(Form)
            self.label_4.setGeometry(QtCore.QRect(20, 440, 251, 16))
            self.label_4.setObjectName(_fromUtf8("label_4"))
     
            self.retranslateUi(Form)
            QtCore.QMetaObject.connectSlotsByName(Form)
     
        def retranslateUi(self, Form):
            Form.setWindowTitle(_translate("Form", "目录探测工具", None))
            self.label.setText(_translate("Form", "网站地址：", None))
            self.button_search.setText(_translate("Form", "探测", None))
            self.label_2.setText(_translate("Form", "结果：", None))
            self.button_add.setText(_translate("Form", "添加地址", None))
            self.label_4.setText(_translate("Form", "例如：/admin/manager.asp  请以斜杠开始", None))

  

2\. 启动文件 start.py

    
    
    #!/usr/local/bin/python
    #coding=utf-8
     
    import sys
    import os
    import time
    import httplib
    import re
    from PyQt4 import QtCore, QtGui
    from threading import Thread
     
    from find_ui import Ui_Form
     
     
    class MyForm(QtGui.QMainWindow):
        def __init__(self, parent=None):
            QtGui.QWidget.__init__(self, parent)
            self.ui = Ui_Form()
            self.ui.setupUi(self)
            QtCore.QObject.connect(self.ui.button_search,QtCore.SIGNAL("clicked()"), self.startthread)
            QtCore.QObject.connect(self.ui.button_add,QtCore.SIGNAL("clicked()"), self.addAddress)
        def startSearch(self):
            self.ui.label_3.setText("") 
            self.getAddress()
            address=str(self.ui.edit_address.text())
            self.accessAddesss(address)
             
        def startthread(self):
            t1=Thread(target=self.startSearch,)
            t1.start()
             
             
        def getAddress(self):
            try:
                global addresslist
                addresslist=[]
                filePath=os.getcwd()+"\\address.txt"
               # if not os.path.isfile(filePath):
                #    print 'aaa'             
                 #   return 0
                    
                fileAddress=file(filePath,"r")
                for address_line in fileAddress.readlines():
                    if address_line not in addresslist:
                        addresslist.append(address_line)
                        pass
                    pass
                pass
                fileAddress.close()
                 
            except:
                #self.ui.text_all.setText('aaa')
                self.ui.text_all.setText(u'打开文件错误')
                pass
            finally:
                #fileAddress.close()
                pass
           # print addresslist[0]
         
        def accessAddesss(self,host):
            try:
                print host
                print len(addresslist)
                for oneAddress in addresslist:
                    print len(addresslist)
                    oneAddress=oneAddress.replace("\n","")
                    print oneAddress
                    connection=httplib.HTTPConnection(host,80,timeout=10)
                    connection.request("GET",oneAddress)
                    response=connection.getresponse()
                    result=response.reason
                    resultNum=response.status
                     
                    if "OK" in result or "Forbidden" in result:
                        getaddress="http://"+host+oneAddress+"------"+str(resultNum)+":"+result
                        self.ui.text_exist.append(getaddress)
                    else:
                        self.ui.text_all.append("http://"+host+oneAddress+"------"+str(resultNum)+":"+result)
                         
                    connection.close()
            except Exception as e:
                print e.message
            self.ui.label_3.setText(u"探测完成") 
            self.ui.label_3.colorCount()
            
        def  addAddress(self):
            try:
                filePath=os.getcwd()+"\\address.txt"      
                fileAddress=file(filePath,"a")
                newAddress="\n"+str(self.ui.edit_add.text())
                print newAddress
                fileAddress.write(newAddress)
                fileAddress.close()
            except Exception as e:
                print e.message
             
     
    if __name__ == "__main__":
        app = QtGui.QApplication(sys.argv)
        myapp = MyForm()
        myapp.show()
        sys.exit(app.exec_())

  

3.address.txt 扫描地址名单文件，可以通过编辑改文件制定自己的规则,你懂的~~

/admin.php

/admin/

/administrator/

/moderator/

/webadmin/

/adminarea/

/bb-admin/

/adminLogin/

/test/login.jsp

/source/login.php

  

