---
title: 超详细Windows + Python + PyQt5 + Pycharm 环境搭建
categories:
  - Qt
tags:
  - Qt
  - Python
  - Windows
  - PyQt5
  - Pycharm
date: 2018-08-06 17:57:13
---

<Excerpt in index | 摘要> 
超详细Windows + Python + PyQt5 + Pycharm 环境搭建，走过路过不要错过~~ <!-- more -->
<The rest of contents | 余下全文>

> 之前一直用Qt写C++，最近想玩玩Python，参考了好几个教程，每个教程都有点坑。于是自己整理了下可能会碰到的问题，方便其他想用PyQt5 (GPL) 的小伙伴吧。如果想用PySide2 (LGPL) 过程也基本类似，就把PyQt5的相关内容替换下就行。

### Python安装

#### Python主程序安装

##### 单个版本Python
[Python官网下载地址](https://www.python.org/downloads/)，目前最新的是Python3.7，我用的是Python3.6，没用最新的主要是担心PyQt那边没更新

##### 多个版本Python
有的小伙伴可能需要安装多个版本的Python，在Linux和Mac上可能需要啥管理工具啥的，具体可以看看[Python版本管理](https://www.jianshu.com/p/60f361822a7e)。**如果你是Windows，直接安装就可以了，具体要用哪个版本的Python只要加上对应的环境变量即可。**
![添加环境变量](https://upload-images.jianshu.io/upload_images/2756183-6d6693a9ce151157.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

也不用装什么版本管理工具，就是这么简单。还有也可以[通过命令行切换Python版本](https://www.zhihu.com/question/21653286)，但是敲命令行有时候老是忘记，不折腾自己了，可以了解下。

#### 给Python添加镜像源
不管是用npm还是pip等包管理工具的时候，国内最好都是换下相关的镜像源，要不然你就只能龟速下载了...

##### 可用镜像源
```
清华：https://pypi.tuna.tsinghua.edu.cn/simple
阿里云：http://mirrors.aliyun.com/pypi/simple/
中国科技大学：https://pypi.mirrors.ustc.edu.cn/simple/
华中理工大学：http://pypi.hustunique.com/
山东理工大学：http://pypi.sdutlinux.org/
豆瓣：http://pypi.douban.com/simple/
```
##### 添加pip.ini
在`C:\Users\Administrator`下新建一个`pip`文件夹，在文件夹中新建一个pip.ini文件：
![创建pip文件夹](https://upload-images.jianshu.io/upload_images/2756183-3c225e0add0beefd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在pip.ini中添加以下内容，之后再用pip下载包就可以体验飞速下载了：
![添加镜像源](https://upload-images.jianshu.io/upload_images/2756183-2575beafce4e786d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### PyQt5安装
环境变量加上了Python对应的文件夹后，就可以通过pip安装PyQt5。默认安装最新的版本，在命令行输入：
```
pip install PyQt5
```
如果需要安装指定版本的QyQt5，改成输入：
```
pip install PyQt5==5.10.1
```


### 界面工具安装
PyQt5只用到了Qt的designer.exe，这是用来设计界面的一个工具，生成的界面文件是.ui的，PyQt5的作用就是把这个ui文件转换成py文件。

#### pyqt5-tools
PyQt5有对应的[这个工具](https://pypi.org/project/pyqt5-tools/)，包含designer.exe。但是官方只更新到了Qt5.9的，而且我也装有Qt，就不考虑此方案了。

#### Qt安装
安装哪个版本基本上都没啥差别，只要有designer.exe，可以生成ui文件就行。附上[Qt下载地址](https://download.qt.io/)。

#### Pycharm安装
这个公司有很多IDE都挺好用的，而且还有一个非常好的福利，**对于有教育邮箱的用户，可以免费使用一年**。附上[通过教育邮箱激活Pycharm](https://www.imsxm.com/2018/02/jetbrain-education-license.html)。

##### Pycharm配置
打开Pycharm，Files->Settings->External Tools，添加两个tools，Qt Designer、PyUIC：
![添加Tools](https://upload-images.jianshu.io/upload_images/2756183-f195d7d5596015f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![添加Qt Designer](https://upload-images.jianshu.io/upload_images/2756183-53a07e1bd2360e95.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. `Programs:`的地方找到你自己的designer.exe所在的位置
2. `Working directory:`的地方填上`$ProjectFileDir$` (填`$FileDir$`好像也行)。

![添加PyUIC](https://upload-images.jianshu.io/upload_images/2756183-442bc6377f6e2f62.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. `Programs:`的地方找到你自己的python.exe所在的位置
2. `Arguments:`的地方填上`-m PyQt5.uic.pyuic  $FileName$ -o $FileNameWithoutExtension$.py`
3. `Working directory:`的地方填上`$FileDir$`

#### Hello World
环境搭建好了，来实际编写一个项目试试

##### 新建Python项目
![新建Python项目](https://upload-images.jianshu.io/upload_images/2756183-a403b6174c1098ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里注意，上面的New environment using是在项目下添加一个Python的环境，如果要使用PyQt5的话要将底下两个勾选框勾上；也可以使用自己本地的Python，就选择底下的Existing interpreter，指定Python.exe位置即可。

##### 创建ui文件
![创建ui文件](https://upload-images.jianshu.io/upload_images/2756183-6087c27e9feb5a96.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

按照图示选择Qt Designer就会启动Qt的designer.exe了，你也可以直接到Qt的目录下运行程序。然后就是愉快的拖控件时间：
![ui文件生成](https://upload-images.jianshu.io/upload_images/2756183-6cba3a2246c67322.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

界面设计完毕，将ui文件保存到Python工程所在的目录下即可，PyCharm中的项目可以看到对应的ui文件。

##### ui文件生成py文件
在ui文件上点击右键，External Tools->PyUIC，就能生成对应的py文件了：
![生成py文件](https://upload-images.jianshu.io/upload_images/2756183-210efd90455b6ed6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 添加main.py
给程序添加一个入口，在工程目录下新建一个main.py，在其中添加代码：
```
import sys
import HelloWorld
from PyQt5.QtWidgets import QApplication, QMainWindow
if __name__ == '__main__':
    app = QApplication(sys.argv)
    MainWindow = QMainWindow()
    ui = HelloWorld.Ui_MainWindow()
    ui.setupUi(MainWindow)
    MainWindow.show()
    sys.exit(app.exec_())
```

![main.py](https://upload-images.jianshu.io/upload_images/2756183-0de8952a9dd4b3d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 运行
在main.py中，点击右键->Run main，即可运行程序。
![运行](https://upload-images.jianshu.io/upload_images/2756183-922da50c66b0251e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![运行界面](https://upload-images.jianshu.io/upload_images/2756183-d58441e8fd4a7bb5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 总结
搭建PyQt5的相关环境其实也不复杂，这其中有很多可以根据自己的环境修改的地方，比如：
- 你需要用PySide2，就换掉PyQt5的内容
- 你有Qt的话，就不用再额外安装designer.exe
- IDE也不一定用PyCharm，你用[VS2017写Python](https://blog.csdn.net/m0_37606112/article/details/78675610)也行

#### 参考链接
- [Python3.6 PyQt5 pycharm 环境搭建](https://www.jianshu.com/p/344bdf61e69e)
- [Python3 PyQt5 pycharm 环境搭建](https://www.jianshu.com/p/094928ac0b73)
- [PIP安装模块下载慢或者无法下载](https://www.qcgzxw.cn/2789.html)
- [PyCharm安装及使用](https://www.jianshu.com/p/042324342bf4)
- [申请JetBrains学生免费注册码](https://www.imsxm.com/2018/02/jetbrain-education-license.html)
- [PyCharm 无法识别PyQt5的两种解决办法](https://blog.csdn.net/leemboy/article/details/80490675)
- [宇宙最强VisualStudio2017配置pyQt5用于python3.6的UI界面工具](https://blog.csdn.net/m0_37606112/article/details/78675610)
