---
title: Qt-QSS
categories:
  - Qt
tags:
  - Qt
  - QSS
date: 2018-05-24 17:49:06
---

<Excerpt in index | 摘要> 
Qt常用QSS集合<!-- more -->
<The rest of contents | 余下全文>

> Qt拿来画控件还是很方便的，其中除了重写paint() 函数外，最常用的就是控件的样式表qss了。本文简单介绍下QSS，同时记录下各种QSS日常骚操作。

## QSS介绍

### QSS简介
QSS称为Qt Style Sheets也就是Qt样式表，它是Qt提供的一种用来自定义控件外观的机制。QSS大量参考了CSS的内容，只不过QSS的功能比CSS要弱很多，体现在选择器要少，可以使用的QSS属性也要少很多，并且并不是所有的属性都可以用在Qt的所有控件上。

### QSS的语法规则
基本和CSS相同，只是有的属性QSS中没有，相当于CSS的子集。

### QSS的选择器类型
| 选择器 | 栗子 | 说明 |
| ---------- | --- | --- |
| 通配选择器 | * | 匹配所有的控件 |
| 类型选择器 | QPushButton | 匹配所有QPushButton和其子类的实例 |
| 属性选择器 | QPushButton[flat="false"] | 匹配所有flat属性是false的QPushButton实例，注意该属性可以是自定义的属性，不一定非要是类本身具有的属性 |
| 类选择器 | .QPushButton | 匹配所有QPushButton的实例，但是并不匹配其子类 |
| ID选择器 | #myButton | 匹配所有id为myButton的控件实例，这里的id实际上就是objectName指定的值 |
| 包含选择器 | QDialog QPushButton | 所有QDialog容器中包含的QPushButton，不管是直接的还是间接的 |
| 子选择器 | QDialog > QPushButton | 所有QDialog容器下面的QPushButton，其中要求QPushButton的直接父容器是QDialog |

另外上面所有的这些选择器可以联合使用，并且支持一次设置多个选择器类型，用逗号隔开，同CSS一样。

### QSS子控件
QSS为很多复杂的复合控件提供了子控件的定义，以方便对这些复合控件的各个部分进行样式设置，具体使用下面说明。

### QSS伪状态
QSS的伪状态选择器实际上与CSS中的类似，是以冒号开头的一个选择表达式，他限制了当控件在某一种状态下的时候才能应用QSS规则，伪状态只能描述一个控件的状态，或者是一个复合控件中的子控件的状态。下面列出常用的伪状态：

| 状态 | 说明 |
| --- | --- |
| :active | 当小部件驻留在活动窗口中时，将设置此状态 |
| :checked | 该控件被选中时候的状态 |
| :hover | 鼠标在控件上方 |
| :pressed | 该控件被按下时的状态 |
| :disabled | 该控件禁用时的状态 |
| :first | 该控件是第一个（列表中） |
| :focus | 该控件有输入焦点时 |
| :has-children | 该控件有孩子。例如，QTreeView中包含子项目的控件 |
| :open | 该控件处于打开状态。例如，QTreeView中的扩展项，或打开菜单的QComboBox或QPushButton |
| :unchecked | 该控件未被选中 |

## QSS代码片段
### 单个控件
```
QPushButton{        /*按钮上图标下文字*/
    text-align:bottom ;
    image-position:top;
    color: #666666;
    border:none;
    image: url(:xxxxx.png);
}

QPushButton{         /*按钮自适应拉伸背景*/
border-width: 2px 15px 2px 15px;
border-image: url(:/resource/icon/button_nomal.png) 2 15 2 15 repeat stretch;
color:#3288E8;
}

QPushButton{        /*按钮选中带下划线*/
border:none;
border-width:5;
border-bottom-style:solid;
border-color:transparent;
color:#666666;
}
QPushButton:checked{
color:#3288E8;
border-color: #3288E8;
}

QPushButton{      /*按钮左文字右图标*/
	background-image:url(:/resource/btn.png);
	background-repeat:norepeat;
	background-position:right;
	text-align:right;
	border:none;
	color:white;	
	padding-right:35px;
}



```

### 含子控件
```
/**************************************CheckBox*********************************************/
QCheckBox::indicator:unchecked{
image: url(:/resource/chebox_off.png)
}

QCheckBox::indicator:checked{
image: url(:/resourcechebox_on.png)
}
/**************************************CheckBox*********************************************/


/**************************************QTabBar*********************************************/
QTabBar::tab {
    color: #666666; 
    height:40px; 
    width:100px;
} 
QTabBar::tab:selected {
    color: #333333; 
    font-weight:bold
}
 QTabBar::tab:hover {
    color: #666666;
}
/**************************************QTabBar*********************************************/


/**************************************滑动条*********************************************/
QSlider {
    background: rgb(170, 170, 170);
    padding: 2px;
    height: 40px;
}
QSlider::groove:horizontal {
    subcontrol-origin: content;
    background: rgb(76, 76, 76);
    
    /* the groove expands to the size of the slider by default. 
    by giving it a height, it has a fixed size */
    height: 20px;
}
QSlider::handle:horizontal {
    background-color: rgb(50, 50, 50);
    width: 40px;
    border-radius: 20px;
    
    /* handle is placed by default on the contents rect of the groove. 
    Expand outside the groove */
    margin: -10px 0;
}
QSlider::sub-page:horizontal {
    background: #999;
    margin: 5px;
    border-radius: 5px;
}
QSlider::add-page:horizontal {
    background: #666;
    margin: 5px;
    border-radius: 5px;
}
/**************************************滑动条*********************************************/


/**************************************下拉框*********************************************/
QComboBox{
border:0px; 
background-color:transparent;
color:#333333; 
}
QComboBox::down-arrow{
image:none;
}
QComboBox::drop-down{
width:0px; 
border:0px;
}
QComboBox QAbstractItemView {
color:#333333;
border: 1px solid #DDDDDD;
background-color:white;
selection-color:white;
selection-background-color: rgb(0, 102, 96);
}
/**************************************下拉框*********************************************/


/**************************************滚动条*********************************************/
QScrollBar:horizontal {
    height: 16px;
    border-width: 0px 10px 0px 10px;
    border-image: url(:/img/horizontal-track.png) 0 10 0 10 repeat stretch;
    margin-left: 6px;
    margin-right: 16px;
    padding-right: 4px;
}
QScrollBar::handle:horizontal {
    min-width: 40px;
    border-width: 0 17px 0 17px;
    border-image: url(:/img/horizontal-handle.png) 0 17 0 17 repeat repeat;
}
QScrollBar::sub-line:horizontal {
    width: 20px;
    height: 17px;
    subcontrol-position: left;
    subcontrol-origin: margin;
    background-image: url(:/img/horizontal-sub-line.png)
}
QScrollBar::add-line:horizontal {
    width: 20px;
    height: 17px;
    subcontrol-position: right;
    subcontrol-origin: border;
    background-image: url(:/img/horizontal-add-line.png)
}
QScrollBar:vertical {
    width: 16px;
    border-width: 10px 0px 10px 0px;
    border-image: url(:/img/vertical-track.png) 10 0 10 0 repeat repeat;
    margin-top: 6px;
    margin-bottom: 16px;
    padding-bottom: 6px;
}
QScrollBar::handle:vertical {
    min-height: 40px;
    border-width: 17px 0px 17px 0px;
    border-image: url(:/img/vertical-handle.png) 17 0 17 0 repeat repeat;
}
QScrollBar::sub-line:vertical {
    width: 17px;
    height: 22px;
    subcontrol-position: top left;
    subcontrol-origin: margin;
    background-image: url(:/img/vertical-sub-line.png)
}
QScrollBar::add-line:vertical {
    width: 17px;
    height: 22px;
    subcontrol-position: bottom;
    subcontrol-origin: border;
    background-image: url(:/img/vertical-add-line.png)
}
/**************************************滚动条*********************************************/
```

持续更新ing...

## 参考资料
- [QSS总结以及最近做的Qt项目](http://www.cnblogs.com/wangqiguo/p/4960776.html)
- [Qss](https://qtdebug.com/qtbook-qss/)
- [Qt之QSS（样式表语法）](https://blog.csdn.net/liang19890820/article/details/51691212)
