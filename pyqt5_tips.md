---
title: 细数PyQt5中遇到的坑
author: zekyou
categories: 计算机
tags:
 - PyQt
 - 攻略
 - Python
date: 2020-02-14
---

​		之前因为说要搞一个自制的文献管理的软件，所以捣鼓了两下PyQt，遇到了不少的坑，遂作此记录，以来备忘，二来分享。<!--more-->

# 起步

## 安装PyQt5

```bash
pip3 install pyqt5 -i https://pypi.tuna.tsinghua.edu.cn/simple
```

## 配合PyCharm

在PyCharm里添加两个工具，方便开发。

- Qt Designer #拖控件式操作，类似AndroidStudio，方便前期初步定下框架

- PyUic #将在Designer里做好的框架直接编译成python，方便后续编辑美化

  添加方法如下：

> File-->Settings-->Tools-->External Tools

点击加号，弹出如下：以PyUIC为例

![pyqt_pyuic](pyqt_pyuic.png)

Program就是程序所在地，Arguments填入宏

> \$FileName\$ -o \$FileNameWithoutExtension\$.py
>
> working建议填入：
>
> \$FileDir$

Designer则只需要Program填写。

# QMainWindow

起初仿照网上的教程，写的是QWidget，结果发现不能添加MenuBar，设置Icon等操作。所以主界面还是要用mainWindow。

```python
        Form.setWindowIcon(QIcon('main.ico'))#set Icon
        Form.setStyleSheet('QMainWindow{border-image:url(bg2.jpg);}')# set background picture
        Form.setWindowOpacity(0.85) #set UI(all) opacity
```

之后就是在PyUIC转化后的代码后面加上：

```python
if __name__=="__main__":
    import sys
    app=QtWidgets.QApplication(sys.argv)
    mainForm=QtWidgets.QMainWindow()
    ui=Ui_mainForm()
    ui.setupUi(mainForm)
    mainForm.show()
    sys.exit(app.exec_())
```

## MenuBar和StatusBar

menubar就是我们使用软件最上面的一栏，

> File	Edit	Tools	Help

之类的，我们希望能在这上面增加功能

在mainwindow里设置menubar和statusbar如下：

```python
        self.menubar = QtWidgets.QMenuBar(MainWindow)
        self.menubar.setGeometry(QtCore.QRect(0, 0, 1003, 26))
        self.menubar.setObjectName("menubar")
        MainWindow.setMenuBar(self.menubar)
        self.statusbar = QtWidgets.QStatusBar(MainWindow)
        self.statusbar.setObjectName("statusbar")
        MainWindow.setStatusBar(self.statusbar)
```



### 打开/创建文件功能

引入

```python
from PyQt5.QtWidgets import QFileDialog
```

打开文件是在menubar上的操作，所以

```python
        self.menuFile = QtWidgets.QMenu(self.menubar)
        self.menuFile.setObjectName("menuFile")
        self.menubar.addAction(self.menuFile.menuAction())
```

我们希望是上面有个File，点一下，出现两个选项，一个New，一个Open

添加

```python
        self.actionNew = QtWidgets.QAction(Form)#new action
        self.actionNew.setObjectName("actionNew")
        self.actionOpen = QtWidgets.QAction(Form)
        self.actionOpen.setObjectName("actionOpen")
        self.menuFile.addAction(self.actionNew)#link action with menu
        self.actionOpen.triggered.connect(self.action_open)#menu use triggered to link
        self.menuFile.addAction(self.actionOpen)
        self.actionNew.triggered.connect(self.action_new)
```

Define action_new and action_open

```python
    def action_new(self):
        try:
            fname,_ = QFileDialog.getSaveFileName(None,'Save File',os.getcwd(),'All Files(*);;NumPy Files(*.npy)','NumPy Files(*.npy)')
            self.textBrowser_14.setText(fname)
        except:
            self.textBrowser_15.setText('Error in saving file')
            
            
    def action_open(self):
        try:
            fname,_ = QFileDialog.getOpenFileName(None,'Select File',os.getcwd(),'All Files(*);;NumPy Files(*.npy)','NumPy Files(*.npy)')
            self.textBrowser_14.setText(fname)
        except:
            self.textBrowser_15.setText('Error in opening file')
```

​		因为QFileDialog没有找到创建，所以使用了save，但效果是一样的，

> os.getcwd() #获取当前目录

fname是文件名（含路径），第二个参数是类型，这里不需要所以用_省略了，函数里的第一个参数确定有无父级，网上很多是self，不过我测试的self，程序就崩溃，所以使用None。

### 状态栏

让状态栏显示文字使用

```python
self.statusbar.showMessage(string,time)#the unit of time is ms(1/1000s)
```

# 快捷键

快捷键定义需要在retranslateUi函数后面，不然一些非组合键的快捷键会失效

```python
self.xxx.setShortcut('ctrl+w')
```



# TextBrowser

一般用的多的就是传递字符串

```python
self.textBrowser.setText(string)
self.textBrowser.toPlainText()#important
```

# LineEdit

字符串传递使用

```python
self.lineEdit.text()
```

获取焦点/光标

```python
self.lineEdit.clearFocus()
self.lineEdit.setFocus()
```

将字符转化为数字

由于text方法得到是str，有时需要参与数值计算，需要数字类型

假设每个数字字符串间是空格

```python
str=self.lineEdit.text()
str=str.split()
#first method
num =[int(x) for x in str]
#second for py2.x
num =map(int, str)
#third for py3.x
num =list(map(int, str))
#forth
num =np.array(str).astype(int)
```



# pushButton

链接槽函数

```python
self.pushButton.clicked.connect(self.xxx)#不带参数
self.pushButton.clicked.connect(lambda:self.xxx(para))#带参数
```

# radioButton

初始默认

```python
self.radioButton.setChecked(True)
```

点击

```python
self.radioButton.clicked.connect(self.xxx)
self.radioButton.clicked.connect(lambda:self.xxx(para))
```

判断是否选择

```python
self.radioButton.isChecked() #boolean
```

# QComboBox

一般用的就是几个：

```python
self.combobox.currentIndex()
self.combobox.currentText()
self.combobox.currentIndexChanged.connect(self.xx)
self.itemText(i) #index=i str
```

# QCheckBox

```python
self.checkbox.ischecked()#boolean
```

# TIPS

## 分离扩展名和路径

```python
import os
dir = self.xxxx.text()
txtdir = os.path.splitext(dir)[0]+'.txt'
```

