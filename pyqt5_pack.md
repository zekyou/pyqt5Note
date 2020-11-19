---
title: PyQ5打包exe
author: zekyou
categories: 计算机
comments: true
date: 2020-02-10
tags:
 - PyQt5
 - Python
---



写好了python代码，如何打包成可执行的exe软件是需要考虑的问题。这里我们使用pyinstaller进行打包。

## 安装pyinstaller

```shell
pip install pyinstaller
```

网速慢的话可以换源，教程[Link](https://zekyou.github.io/2020/02/05/pip_change_rs/)<!--more-->

## 打包

假设主程序是main.py

引用的自定义第三方类需要在同一目录或者引用添加路径

键入

```shell
pyinstaller main.py #最直接简单
#就可以在该目录下的/dist中生成一个main文件夹，注意是文件夹，这个文件夹内有一个main.exe。需要传给别人时，要将整个目录copy过去
pyinstaller -F main.py
#加了一个-F参数，区别是只生成一个main.exe文件，同样在/dist目录下。不过这个main.exe比上面那个大很多，但是整体来讲比main目录要小。
pyinstaller -F main.py -i xxx.ico -w
#这个加了两个参数，-i是指定生成exe文件的图标（但不是运行后的图标） -w则是免去执行exe时的命令行弹窗。我们设计软件如果不是命令行界面就需要加入这个参数了，不然启动软件回先弹出一个命令行窗口，有点丑。
```

## 遇到的问题

如果就是这么顺利，那也不会写个博客。在win系统下打包会出现各种各样的问题，主要还是windows系统环境变量的问题。比如：

### 出现No module named xxx(bs4)

这种一般是缺失了某个库，直接安装

```shell
pip install xxx
```

但大部分情况是，其实你安装了却还是出错。

```shell
pip list #检测安装了哪些库
```

这时候就需要你指定库安装的路径了，一般是python的安装目录下/lib/site-package

此时键入

```shell
pyinstaller main.py -F -i xxx.ico -w -p xxx/python/lib/site-package 
#-p 指定路径
```

### 代码内部引用资源

一般做软件会自定义一些自己的图标，背景图片，或是控件图标。但是我们打包后这些图标同样需要copy到main.exe相应的路径下。不然会出现缺失。



## Spec文件

spec文件是伴随打包而出现的文件，相当于是对main.py打包的配置文件，我们之前输入的那么多参数就体现在spec文件内，以此如果我们不想在输入这么多的参数了，就可以直接去spec文件内修改

```shell
vim main.spec
```

之后键入

```shell
pyinstaller main.spec
```

这就算完成了